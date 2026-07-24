---
title: "Zed"
date: 2026-07-24
tags:
  - config
  - zed
publish: false
---

# Zed

## settings.json

```json
// Zed settings
//
// For information on how to configure Zed, see the Zed
// documentation: https://zed.dev/docs/configuring-zed
//
// To see all of Zed's default settings without changing your
// custom settings, run `zed: open default settings` from the
// command palette (cmd-shift-p / ctrl-shift-p)
{
  "base_keymap": "Cursor",
  "ui_font_family": "Inter",
  "buffer_line_height": "comfortable",
  "redact_private_values": true,
  "use_system_window_tabs": false,
  "title_bar": {
    "show_branch_status_icon": true,
    "show_menus": false,
  },
  "tabs": {
    "file_icons": true,
    "git_status": true,
  },
  "disable_ai": false,
  "agent": {
    "default_profile": "write",
    "default_model": {
      "provider": "copilot_chat",
      "model": "gpt-5-mini",
    },
    "model_parameters": [],
  },
  "edit_predictions": {
    "mode": "eager",
  },
  "icon_theme": {
    "mode": "system",
    "light": "Zed (Default)",
    "dark": "Zed (Default)",
  },
  "terminal": {
    "font_family": "JetBrainsMono Nerd Font",
  },
  "project_panel": {
    "auto_fold_dirs": false,
  },
  "status_bar": {
    "experimental.show": false,
  },
  "tab_bar": {
    "show": true,
  },
  "show_signature_help_after_edits": false,
  "auto_signature_help": false,
  "sticky_scroll": {
    "enabled": false,
  },
  "gutter": {
    "line_numbers": true,
  },
  "minimap": {
    "show": "never",
  },
  "current_line_highlight": "gutter",
  "cursor_shape": "block",
  "cursor_blink": true,
  "autosave": "on_window_change",
  "buffer_font_family": "JetBrainsMono Nerd Font",
  "colorize_brackets": false,
  "auto_indent_on_paste": true,
  "show_whitespaces": "none",
  "ensure_final_newline_on_save": true,
  "indent_guides": {
    "enabled": false,
  },
  "soft_wrap": "editor_width",
  "ui_font_size": 16.0,
  "buffer_font_size": 16.0,
  "theme": {
    "mode": "system",
    "light": "Ayu Light",
    "dark": "Ayu Dark",
  },
}
```

## Related

- [[visual-studio-code]]
- [[vi-motions]]
