---
title: "Visual Studio Code"
date: 2026-07-24
tags:
  - config
  - vscode
publish: false
---

# Visual Studio Code Configuration

## settings.json

```json
{
  // === Appearance ===
  "window.title": "",
  "window.customTitleBarVisibility": "auto",
  "window.commandCenter": false,
  "workbench.activityBar.location": "hidden",
  "workbench.editor.showTabs": "single",
  "workbench.statusBar.visible": false,
  "workbench.sideBar.location": "right",
  "workbench.layoutControl.enabled": false,
  "workbench.tips.enabled": false,
  "window.zoomLevel": 1,

  // === Editor: Font & Display ===
  "fonted.font": "Inter",
  "editor.fontFamily": "JetBrainsMono Nerd Font Mono",
  "debug.console.fontFamily": "JetBrainsMono Nerd Font Mono",
  "editor.cursorBlinking": "smooth",
  "editor.cursorStyle": "block",
  "editor.wordWrap": "on",
  "editor.fontSize": 14,
  "editor.matchBrackets": "near",
  "editor.renderLineHighlight": "gutter",
  "editor.cursorSmoothCaretAnimation": "on",
  "editor.smoothScrolling": true,
  "editor.snippetSuggestions": "top",

  "editor.minimap.enabled": true,
  "editor.lineNumbers": "on",
  "editor.glyphMargin": true,
  "editor.guides.indentation": false,
  "editor.renderWhitespace": "none",
  "editor.hideCursorInOverviewRuler": true,
  "editor.bracketPairColorization.enabled": false,
  "editor.colorDecorators": false,
  "editor.occurrencesHighlight": "off",
  "editor.stickyScroll.enabled": false,

  // === Editor: Formatting & Suggestions ===
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.indentSize": "tabSize",
  "editor.quickSuggestions": {
    "comments": true,
    "strings": true,
    "other": true,
  },
  "editor.parameterHints.enabled": false,
  "editor.suggest.snippetsPreventQuickSuggestions": false,
  "editor.accessibilitySupport": "off",

  // === Files & Explorer ===
  "files.insertFinalNewline": true,
  "files.autoSave": "onWindowChange",
  "explorer.confirmPasteNative": false,
  "explorer.confirmDelete": false,
  "explorer.confirmDragAndDrop": false,
  "explorer.compactFolders": false,
  "explorer.decorations.badges": false,
  "explorer.openEditors.visible": 1,

  // === Terminal ===
  "terminal.integrated.smoothScrolling": true,

  // === Language Specific ===
  "tailwindCSS.includeLanguages": {
    "blade": "html",
  },
  "typescript.updateImportsOnFileMove.enabled": "always",
  "javascript.updateImportsOnFileMove.enabled": "always",

  // === Language Formatters ===
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },

  // === Git stuff ===
  "diffEditor.ignoreTrimWhitespace": false,
  "diffEditor.renderSideBySide": false,
  "git.confirmSync": false,
  "git.autofetch": true,
  "git.enableSmartCommit": true,

  // === Misc ===
  "symbols.hidesExplorerArrows": false,
  "workbench.colorCustomizations": {},
  "editor.inlayHints.fontFamily": "JetBrainsMono Nerd Font Mono",
  "notebook.markup.fontFamily": "JetBrainsMono Nerd Font Mono",
  "gitlens.currentLine.fontFamily": "JetBrainsMono Nerd Font Mono",
  "gitlens.blame.fontFamily": "JetBrainsMono Nerd Font Mono",
  "markdown.preview.fontFamily": "JetBrainsMono Nerd Font Mono",
  "terminal.integrated.fontLigatures.enabled": true,
  "debug.disassemblyView.showSourceCode": false,
  "prisma.hidePrisma6Prompts": true,
  "git.openRepositoryInParentFolders": "always",
  "window.autoDetectColorScheme": false,
  "editor.codeLensFontFamily": "JetBrainsMono Nerd Font Mono",
  "workbench.colorTheme": "Vesper",
  "workbench.iconTheme": "symbols",
  "workbench.agentsWindowButton.enabled": false,
}
```

## How to override Visual Studio Code default font styling

https://github.com/microsoft/vscode/issues/519#issuecomment-3691806331

## Related

- [[zed]]
- [[vi-motions]]
