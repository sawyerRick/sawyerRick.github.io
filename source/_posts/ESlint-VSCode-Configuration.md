---
title: ESlint + VSCode Configuration
date: 2022-09-10 23:23:28
tags: Frontend
---

# ESlint + VSCode Configuration

Requirement: I want whenever I press `Command + S` ESLint will do an all-fix in current file automatically.

Here is a tooling list just for that requirement.

## VSCode Plugins

- ESLint

## Npm Dependencies

- npm init @eslint/config

## VSCode Configurations

All you need to do is to add only one line to `settings.json`

```json
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```

## Ending

Have you set up toolings mentioned above, ESLint wil do an all-fix in current file automatically when you press `Command + S`.

That's it, simple as that.
