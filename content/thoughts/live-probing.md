---
title: "Live Probing"
date: 2026-07-24
tags:
  - work
  - live-probing
publish: false
---

# Live Probing

## Deploy Scripts

### deploy.js

```javascript
#!/usr/bin/env node
/**
 * Deploy LiveProbe to a remote server over SSH/SFTP.
 *
 * Usage:
 *   node deploy.js
 *   npm run deploy
 *
 * Prompts interactively for connection details and deploy options.
 * pm2 start/restart on the server is optional (you can skip it at the prompt).
 */

import { execSync } from "node:child_process";
import { createReadStream } from "node:fs";
import { readdir, stat } from "node:fs/promises";
import { dirname, join, posix, relative } from "node:path";
import { createInterface } from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";
import { fileURLToPath } from "node:url";
import { Client } from "ssh2";

const ROOT = dirname(fileURLToPath(import.meta.url));

const DEFAULTS = {
  username: "root",
  sshPort: "22",
  remotePath: "/var/www/live-probing",
  appPort: "4319",
  collectorPort: "4320",
  restart: true,
  skipBuild: false,
  withCollector: true,
};

/**
 * @typedef {Object} DeployOptions
 * @property {string} host
 * @property {string} password
 * @property {string} username
 * @property {number} sshPort
 * @property {string} remotePath
 * @property {string} appPort
 * @property {string} collectorPort
 * @property {boolean} restart
 * @property {boolean} skipBuild
 * @property {boolean} withCollector
 */

/** @param {string} question */
function askPassword(question) {
  return new Promise((resolve) => {
    output.write(question);
    input.resume();
    input.setRawMode(true);
    input.setEncoding("utf8");

    let password = "";
    /** @param {string} char */
    const onData = (char) => {
      if (char === "\n" || char === "\r" || char === "\u0004") {
        input.setRawMode(false);
        input.removeListener("data", onData);
        output.write("\n");
        resolve(password);
        return;
      }
      if (char === "\u0003") {
        process.exit(130);
      }
      if (char === "\u007f" || char === "\b") {
        if (password.length > 0) {
          password = password.slice(0, -1);
          output.write("\b \b");
        }
        return;
      }
      password += char;
      output.write("*");
    };

    input.on("data", onData);
  });
}

/** @returns {Promise<DeployOptions>} */
async function promptDeployOptions() {
  const rl = createInterface({ input, output });

  /**
   * @param {string} question
   * @param {string} [defaultValue]
   */
  const ask = async (question, defaultValue) => {
    const suffix = defaultValue !== undefined ? ` [${defaultValue}]` : "";
    const answer = (await rl.question(`${question}${suffix}: `)).trim();
    return answer || (defaultValue ?? "");
  };

  /**
   * @param {string} question
   * @param {boolean} defaultYes
   */
  const askYesNo = async (question, defaultYes) => {
    const defaultLabel = defaultYes ? "Y/n" : "y/N";
    const answer = (await rl.question(`${question} [${defaultLabel}]: `))
      .trim()
      .toLowerCase();
    if (!answer) {
      return defaultYes;
    }
    return answer === "y" || answer === "yes";
  };

  try {
    console.log("LiveProbe deploy\n");

    const host = await ask("Server IP or hostname");
    if (!host) {
      throw new Error("server IP or hostname is required");
    }

    const username = await ask("SSH user", DEFAULTS.username);
    const sshPortRaw = await ask("SSH port", DEFAULTS.sshPort);
    const remotePath = await ask(
      "Remote install directory",
      DEFAULTS.remotePath,
    );
    const appPort = await ask("LiveProbe listen port", DEFAULTS.appPort);
    const withCollector = await askYesNo(
      "Start algo-instrumentation collector with serve.sh (--collector)?",
      DEFAULTS.withCollector,
    );
    let collectorPort = DEFAULTS.collectorPort;
    if (withCollector) {
      collectorPort = await ask(
        "Collector listen port",
        DEFAULTS.collectorPort,
      );
    }
    const skipBuild = await askYesNo(
      "Skip local UI build?",
      DEFAULTS.skipBuild,
    );
    const restart = await askYesNo(
      "Start or restart LiveProbe with pm2 after upload?",
      DEFAULTS.restart,
    );

    rl.close();

    const sshPort = Number(sshPortRaw);
    if (!Number.isInteger(sshPort) || sshPort < 1 || sshPort > 65535) {
      throw new Error("SSH port must be a number between 1 and 65535");
    }

    const listenPort = Number(appPort);
    if (!Number.isInteger(listenPort) || listenPort < 1 || listenPort > 65535) {
      throw new Error(
        "LiveProbe listen port must be a number between 1 and 65535",
      );
    }

    if (withCollector) {
      const collectorListenPort = Number(collectorPort);
      if (
        !Number.isInteger(collectorListenPort) ||
        collectorListenPort < 1 ||
        collectorListenPort > 65535
      ) {
        throw new Error(
          "Collector listen port must be a number between 1 and 65535",
        );
      }
    }

    const password = await askPassword("SSH password: ");
    if (!password) {
      throw new Error("SSH password is required");
    }

    return {
      host,
      password,
      username,
      sshPort,
      remotePath,
      appPort,
      collectorPort,
      restart,
      skipBuild,
      withCollector,
    };
  } catch (err) {
    rl.close();
    throw err;
  }
}

/** @param {string} label */
function step(label) {
  console.log(`\n==> ${label}`);
}

/** @param {import('ssh2').Client} conn @param {string} cmd */
function execRemote(conn, cmd) {
  return new Promise((resolve, reject) => {
    conn.exec(cmd, (err, stream) => {
      if (err) {
        reject(err);
        return;
      }
      let stdout = "";
      let stderr = "";
      stream.on("data", (chunk) => {
        stdout += chunk.toString();
        process.stdout.write(chunk);
      });
      stream.stderr.on("data", (chunk) => {
        stderr += chunk.toString();
        process.stderr.write(chunk);
      });
      stream.on("close", (code) => {
        if (code !== 0) {
          reject(
            new Error(`remote command failed (${code}): ${cmd}\n${stderr}`),
          );
          return;
        }
        resolve(stdout);
      });
    });
  });
}

/** @param {import('ssh2').SFTPWrapper} sftp @param {string} remoteDir */
function mkdirRemote(sftp, remoteDir) {
  return new Promise((resolve, reject) => {
    sftp.mkdir(remoteDir, { mode: 0o755 }, (err) => {
      if (!err || err.code === 4) {
        resolve();
        return;
      }
      reject(err);
    });
  });
}

/** @param {import('ssh2').SFTPWrapper} sftp @param {string} localPath @param {string} remotePath */
function putFile(sftp, localPath, remotePath) {
  return new Promise((resolve, reject) => {
    const stream = createReadStream(localPath);
    const writeStream = sftp.createWriteStream(remotePath, { mode: 0o644 });
    stream.on("error", reject);
    writeStream.on("error", reject);
    writeStream.on("close", resolve);
    stream.pipe(writeStream);
  });
}

/**
 * @param {import('ssh2').SFTPWrapper} sftp
 * @param {string} localDir
 * @param {string} remoteDir
 * @param {(relativePath: string) => boolean} include
 */
async function uploadDir(sftp, localDir, remoteDir, include) {
  await mkdirRemote(sftp, remoteDir);
  const entries = await readdir(localDir, { withFileTypes: true });
  for (const entry of entries) {
    const localPath = join(localDir, entry.name);
    const remoteEntry = posix.join(remoteDir, entry.name);
    const rel = relative(ROOT, localPath);
    if (!include(rel)) {
      continue;
    }
    if (entry.isDirectory()) {
      await uploadDir(sftp, localPath, remoteEntry, include);
      continue;
    }
    if (!entry.isFile()) {
      continue;
    }
    await mkdirRemote(sftp, remoteDir);
    process.stdout.write(`    ${rel}\n`);
    await putFile(sftp, localPath, remoteEntry);
  }
}

/** @param {string} relativePath */
function shouldUpload(relativePath) {
  const normalized = relativePath.replaceAll("\\", "/");
  if (normalized.includes("node_modules/")) {
    return false;
  }
  if (normalized.endsWith(".tsbuildinfo")) {
    return false;
  }
  // The UI is built locally and shipped as static files in packages/server/public,
  // so the server never needs the UI source or its vite/react toolchain.
  if (normalized === "packages/ui" || normalized.startsWith("packages/ui/")) {
    return false;
  }
  return true;
}

function buildUiLocally() {
  step("Building UI locally");
  execSync("npm install --silent", { cwd: ROOT, stdio: "inherit" });
  execSync("npm run build -w @liveprobe/ui", { cwd: ROOT, stdio: "inherit" });
  execSync(
    "rm -rf packages/server/public && cp -r packages/ui/dist packages/server/public",
    {
      cwd: ROOT,
      stdio: "inherit",
      shell: true,
    },
  );
}

/**
 * @param {DeployOptions} options
 * @returns {Promise<import('ssh2').Client>}
 */
function connect(options) {
  return new Promise((resolve, reject) => {
    const conn = new Client();
    conn
      .on("ready", () => resolve(conn))
      .on("error", reject)
      .connect({
        host: options.host,
        port: options.sshPort,
        username: options.username,
        password: options.password,
        readyTimeout: 20000,
      });
  });
}

/** @param {DeployOptions} options */
function pm2ServeCommand(options) {
  const serveCmd = options.withCollector
    ? "./serve.sh --collector"
    : "./serve.sh";
  const envParts = [`PORT=${shellQuote(options.appPort)}`];
  if (options.withCollector) {
    envParts.push(`COLLECTOR_PORT=${shellQuote(options.collectorPort)}`);
  }
  return `${envParts.join(" ")} pm2 start bash --name liveprobe -- -c ${shellQuote(serveCmd)}`;
}

/** @param {DeployOptions} options */
function manualStartHint(options) {
  return options.withCollector
    ? "pm2 start ./serve.sh --collector --name liveprobe --interpreter bash"
    : "pm2 start ./serve.sh --name liveprobe --interpreter bash";
}

/**
 * @param {string} appPort
 * @param {string} [collectorPort]
 */
function remoteHealthChecks(appPort, collectorPort) {
  const checks = [
    `(for i in $(seq 1 15); do curl -sf "http://127.0.0.1:${appPort}/" >/dev/null && exit 0; sleep 1; done; exit 1) && echo "LiveProbe is up on port ${appPort}" || echo "WARNING: LiveProbe health check failed on port ${appPort} — check: pm2 logs liveprobe"`,
  ];
  if (collectorPort) {
    checks.push(
      `(for i in $(seq 1 15); do curl -sf "http://127.0.0.1:${collectorPort}/healthz" >/dev/null && exit 0; sleep 1; done; exit 1) && echo "Collector is up on port ${collectorPort}" || echo "WARNING: collector health check failed on port ${collectorPort} — check: pm2 logs liveprobe (port may be in use or collector still starting)"`,
    );
  }
  return checks.join(" && ");
}

async function main() {
  const options = await promptDeployOptions();
  const {
    host,
    username,
    sshPort,
    remotePath,
    appPort,
    restart,
    skipBuild,
    withCollector,
    collectorPort,
  } = options;

  if (!skipBuild) {
    buildUiLocally();
  } else if (
    !(await stat(join(ROOT, "packages/server/public/index.html")).catch(
      () => null,
    ))
  ) {
    console.error(
      "deploy: skip local UI build was selected but packages/server/public/index.html is missing",
    );
    process.exit(1);
  }

  step(`Connecting to ${username}@${host}:${sshPort}`);
  const conn = await connect(options);

  try {
    const sftp = await new Promise((resolve, reject) => {
      conn.sftp((err, client) => {
        if (err) {
          reject(err);
          return;
        }
        resolve(client);
      });
    });

    step(`Uploading to ${remotePath}`);
    await execRemote(conn, `mkdir -p ${shellQuote(remotePath)}`);

    const topLevelFiles = ["serve.sh", "package.json", "package-lock.json"];
    for (const file of topLevelFiles) {
      const localPath = join(ROOT, file);
      if (!(await stat(localPath).catch(() => null))) {
        throw new Error(`missing local file: ${file}`);
      }
      process.stdout.write(`    ${file}\n`);
      await putFile(sftp, localPath, posix.join(remotePath, file));
    }

    await uploadDir(
      sftp,
      join(ROOT, "packages"),
      posix.join(remotePath, "packages"),
      shouldUpload,
    );

    step("Installing dependencies on server");
    const remoteCmd = [
      `cd ${shellQuote(remotePath)}`,
      "chmod +x serve.sh",
      "npm install --silent",
      restart
        ? [
            'command -v pm2 >/dev/null || { echo "pm2 not found on server; install it first" >&2; exit 1; }',
            "pm2 delete liveprobe 2>/dev/null || true",
            pm2ServeCommand(options),
            "pm2 save",
            remoteHealthChecks(
              appPort,
              withCollector ? collectorPort : undefined,
            ),
          ]
            .filter(Boolean)
            .join(" && ")
        : `echo "Upload complete. pm2 was skipped — SSH in and start manually if needed: ${manualStartHint(options)}"`,
    ].join(" && ");

    await execRemote(conn, remoteCmd);

    console.log("\nDeploy finished.");
    console.log(`  UI + API: http://${host}:${appPort}`);
    if (withCollector) {
      console.log(
        `  Collector: http://${host}:${collectorPort}  (/v1/event/* -> LiveProbe)`,
      );
    }
    console.log("  Keep these ports private — ingest has no auth.");
  } finally {
    conn.end();
  }
}

/** @param {string} value */
function shellQuote(value) {
  return `'${value.replaceAll("'", `'\"'\"'`)}'`;
}

main().catch((err) => {
  console.error(`\ndeploy failed: ${err.message}`);
  process.exit(1);
});
```

### package.json

```javascript
{
  "name": "liveprobe",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "workspaces": [
    "packages/*"
  ],
  "engines": {
    "node": ">=20"
  },
  "scripts": {
    "test": "node --import tsx --test \"packages/*/src/**/*.test.ts\"",
    "test:e2e": "playwright test",
    "e2e:serve": "npm run build -w @liveprobe/ui && rm -rf packages/server/public && cp -r packages/ui/dist packages/server/public && rm -rf .e2e-data && mkdir -p .e2e-data && PORT=4399 DB_PATH=.e2e-data/liveprobe.db npx tsx packages/server/src/index.ts",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "@playwright/test": "^1.49.1",
    "@types/amqplib": "^0.10.6",
    "@types/node": "^22.10.0",
    "@types/ws": "^8.18.1",
    "ssh2": "^1.17.0",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2"
  }
}
```

## Related

- [[qubefini]]
- [[qubefini-server-nginx-backup]]
