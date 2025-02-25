---
title: "⇧⇩ FortuneExcel - Lessons from my First Package"
date: 2024-11-18T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["excel", "fortune-sheet", "npm"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---


I revived an archived project and released it as [FortuneExcel](http://npmjs.com/package/@corbe30/fortune-excel) - a plugin for FortuneSheet to import/export Excel (.xlsx) files. 

<table>
  <tr>
    <td  valign="top"><img src="https://img.shields.io/npm/v/@corbe30/fortune-excel" /></td>
    <td  valign="top"><img src="https://img.shields.io/npm/d18m/%40corbe30%2Ffortune-excel" /> </td>
  </tr>
 </table>

## Initialization

Reviving the project meant fixing the deprecated code and existing bugs. Along with this, I added export feature, toolbar plugin support for fortune-sheet and set up Github workflow.

## Modularization

While implementing a package, it should have maximum abstraction. User should have to add minimal code to their environment for your package to work, all while having no side-effects on their existing code. A proper code strucrure also makes it easier for others to contribute to your code.

## Github Workflow

Lets breakdown the pubhlish.yml file.

Run the workflow whenever a release is published or manually triggered from Actions tab:

```yml
publish.yml
-----------
on:
  release:
    types: [published]
  workflow_dispatch:
```

Define premissions required by the workflow:

```yml
permissions:
  contents: read
  pages: write
  id-token: write
```

Checkout the code, setup node environment, install dependencies (see [npm i vs npm ci](https://stackoverflow.com/questions/52499617/what-is-the-difference-between-npm-install-and-npm-ci)) and publish the package on npm:

```yml
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

```yml
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

```json
// tsconfig.json
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

## Compiling TS for Storybook

To make our plugin work with Storybook in the <i>same repository</i>, we can take two approaches:

1. Using Compiled JavaScript (CommonJS from `/dist/`)
2. Using TypeScript Source Directly

Using TS source directly enables seamless debugging in browsers, but it will require additional configuration. To leverage your TypeScript source directly within Storybook:

1. Configure TypeScript Path Aliases: Modify your tsconfig.json to create a path alias for your plugin's entry point. This allows Storybook to resolve the import correctly.

   ```json
   // tsconfig.json
   {
     "paths": {
        "@corbe30/fortune-excel": ["./src/main.ts"],
     },
     ...
   }
   ```

   ```tsx
   // stories/Page.tsx
   import {
     exportToolBarItem,
     importToolBarItem,
   } from "@corbe30/fortune-excel";
   ```

2. Configure Babel for TypeScript Support: Storybook uses Babel for its build process. To enable TypeScript compilation, you need to add the `@babel/preset-typescript` preset to your Babel configuration (`babel.config.js` or .`babelrc`).

   ```js
   // babel.config.js
   module.exports = {
     presets: [
       "@babel/preset-env",
       "@babel/preset-react",
       "@babel/preset-typescript"
     ],
     ...
   };
   ```

3. Install Babel Presets: Ensure you have the necessary Babel presets installed as development dependencies.
   ```sh
   npm install --save-dev @babel/preset-env @babel/preset-react @babel/preset-typescript
   ```

