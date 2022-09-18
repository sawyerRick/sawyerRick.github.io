---
title: ESlint + VSCode Configuration
date: 2022-09-10 23:23:28
tags: Frontend
---

# ESlint + VSCode Configuration

Requirement: I want whenever I press `Command + S` ESLint will do an all-fix in current file automatically.

Here is a tooling list just for that requirement.

1. VSCode plugins
2. npm dependencies
3. VSCode Configurations

## VSCode Plugins

- ESLint

## Npm Dependencies

Basically all we need is ESLint rule file-'.eslintrc.cjs', so let's run one of the commands below.

- npm init @eslint/config
- npm install eslint && npx eslint --init

## VSCode Configurations

All you need to do is to add only one line to `settings.json`

```json
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```

## My ESLint .eslintrc.cjs

In case I need it, duplicate my .eslintrc.cjs here.

```json
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'plugin:react/recommended',
    'standard'
  ],
  overrides: [
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: [
    'react'
  ],
  rules: {
    'react/prop-types': 0
  }
}
```



## Ending

Once you set up toolings mentioned above, ESLint wil do an all-fix in current file automatically when you press `Command + S`.

That's it, simple as that.
