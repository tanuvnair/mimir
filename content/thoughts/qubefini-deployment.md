---
title: "Qubefini Deployment"
date: 2026-07-24
tags:
  - qubefini
  - work
  - deployment
publish: false
---

# Qubefini Deployment

## Deployment Guide (Staging)

This document describes the steps to deploy the backend and frontend on the staging server. Follow each step in order and verify status and logs at the end.

### Prerequisites

- SSH access to `root@tme.qubefini.com`
- Server password
- PM2 installed on the server
- Git configured on the server

### Pre-Deployment

1. **Notify the team**—Send a message in Space that staging will be down.

### Server Access

1. **SSH into the server:**

```bash
ssh root@tme.qubefini.com
```

1. **Enter the server password** when prompted.

### Backend Deployment

1. **Stop all running processes:**

```bash
pm2 stop all
```

1. **Navigate to the backend directory:**

```bash
cd /var/www/mini-etl/
```

1. **Checkout and pull the latest changes from develop:**

```bash
git checkout develop
git pull origin develop
```

1. **Run the build script:**

Builds binaries for Linux and Windows; outputs to `./bin`

```bash
./scripts/build.sh
```

1. **Deploy Prisma migrations:**

```bash
npx prisma@6 migrate deploy
```

1. **Update environment variables (if applicable):**

- Add new variables to `/var/www/mini-etl/.env`
- Update the backend section in `ecosystem.config.cjs` if using PM2 for environment management

### Frontend Deployment

1. **Navigate to the frontend directory:**

```bash
cd /var/www/mini-etl-ui/
```

1. **Checkout and pull the latest changes from develop:**

```bash
git checkout develop
git pull origin develop
```

1. **Install dependencies:**

```bash
npm install
```

1. **Build the frontend:**

```bash
npm run build
```

1. **Update environment variables (if applicable):**

- Add new variables to the frontend environment file
- Update the frontend section in `ecosystem.config.cjs` so PM2 picks up the new values on restart

### Post-Deployment

1. **Restart all processes:**

```bash
pm2 restart all
```

To refresh environment variables for a specific process:

```bash
pm2 restart <process-id-or-name> --update-env
```

1. **Verify deployment status:**

```bash
pm2 status
pm2 logs
```

### Troubleshooting

| Issue | Solution |
| --- | --- |
| Uncertain about current branch | Run `git status` to verify you're on `develop` |
| Prisma migrations fail | Verify Prisma version and database connection in `.env` |
| Service fails to start | Check logs with `pm2 logs <process-id-or-name>` and verify binaries and environment variables |
| Environment variables not picked up | Ensure variables are in both `.env` and `ecosystem.config.cjs`, then use `pm2 restart <id> --update-env` |

### Best Practices

- Always verify you are on the `develop` branch before pulling
- Create backups of `.env` and `ecosystem.config.cjs` before making edits
- Review `pm2 logs` for any warnings or errors after deployment
- Test critical functionality in staging before promoting to production

## Deployment Guide (Production)

Run the following to give the node package manager the right certificates to install packages

```sh
set NODE_EXTRA_CA_CERTS=E:\Softwares\Zscaler Root CA.crt
```

## Deployment flow improvements

- Test API key implementation on staging and try hitting Qubefini using the API key in a TM1 TI process
- Add a Makefile in qubefini for automated build and release
- Update qubefini-ui to avoid `VITE_`-prefixed variables; route config through server → client where needed
- Add a Makefile in qubefini-ui for automated build and release
- On IG deployment, use a single ecosystem file for frontend and backend so env/ecosystem is not copy-pasted every time
- Create a PAT and try deploying qubefini with it on IG
- Test outbound IP addresses and give them to sir for allowlisting

## Intergold TM1 → Qubefini exposure

- TM1 dashboard UI button runs a TI process; TI calls HTTP (built-in function)
- IG Qubefini is not exposed to the internet yet
- Identify outbound IP addresses of TM1, verify them, then ask IG IT to expose Qubefini to those IPs

## Related

- [[qubefini]]
- [[qubefini-rbac-deployment-steps]]
- [[qubefini-multi-tenant-and-plans]]
- [[qubefini-server-nginx-backup]]
- [[intergold]]
