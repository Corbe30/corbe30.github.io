---
layout: post
title: FortuneExcel - Lessons from my First Package
date: 2024-11-18 10:00:00
categories: [excel, fortune-sheet, npm]
---

I revived an archived project and released it as [FortuneExcel](http://npmjs.com/package/@corbe30/fortune-excel) 
<img src="https://img.shields.io/npm/v/@corbe30/fortune-excel" /> - a plugin for FortuneSheet to import/export Excel (.xlsx) files.

## Initialization

Reviving the project meant fixing the deprecated code and existing bugs. Along with this, I added export feature, toolbar plugin support for fortune-sheet and set up Github workflow.

## Modularization

While implementing a package, it should have maximum abstraction. User should have to add minimal code to their environment for your package to work, all while having no side-effects on their existing code. A proper code strucrure also makes it easier for others to contribute to your code.

## Github Workflow

Lets breakdown the pubhlish.yml file.

Run the workflow whenever a release is published or manually triggered from Actions tab:

```
publish.yml
-----------
on:
  release:
    types: [published]
  workflow_dispatch:
```

Define premissions required by the workflow:

```
permissions:
  contents: read
  pages: write
  id-token: write
```

Checkout the code, setup node environment, install dependencies (see [npm i vs npm ci](https://stackoverflow.com/questions/52499617/what-is-the-difference-between-npm-install-and-npm-ci)) and publish the package on npm:

```
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
    with:
        node-version: "20.x"
        registry-url: "https://registry.npmjs.org"
    - run: npm ci
    - run: npm publish --tag latest --provenance --access public
    env:
        NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Build the Storybook, upload it as an artifact and deploy it on GitHub Pages:

> Note: Update Build and Deployement source to 'Github Actions' in repo > Settings > Pages

```
    - run: npm run build-storybook
    - uses: actions/upload-pages-artifact@v2
    with:
        path: storybook-static
    - id: deploy
    uses: actions/deploy-pages@v3
    with:
        token: ${{ github.token }}
```

## Secrets and Tokens

In the above section inside publish.yml, notice `github.token` and `secrets.NPM_TOKEN`.

While `github.token` is automatically provided by GitHub Actions, `secrets.NPM_TOKEN` is an [_automation_ access token](https://docs.npmjs.com/about-access-tokens) that has to be generated at npmjs.com and [set as repository secret](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository) on Github.

## Emitted Javascript version

Fortune-excel and fortune-sheet were having incompatibility issues due to different transmitted JS when compiled from TS. This is defined by the `target` value in `compilerOptions` of tsconfig.json. Since fortune-sheet compiles the JS In es5, changing the target from es6 to es5 in fortune-excel fixed the issue:

```
tsconfig.json
-----------
{
  "compilerOptions": {
    "target": "es5",
    ...
  }
}
```

## Commonjs entrypoint

After publishing the package, Yarn showed this warning - '_the package doesn't seem to have a commonjs entry point_'. Compiling the TS in commonjs is necessary for old projects still using CJS instead of ESM. Compiling is ESM is optional since EMS supports CJS, but not visa-versa. To fix the warning:

1. Make sure package.json does not have `"type": "module"` defined
   > When set to "module", the type field allows a package to specify all .js files within are ES modules. If the "type" field is omitted or set to "commonjs", all .js files are treated as CommonJS.
2. Define `"module": "commonjs"` in tsconfig.json - refer to [this](https://www.typescriptlang.org/tsconfig/#module).


