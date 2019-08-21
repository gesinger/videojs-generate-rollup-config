# videojs-generate-rollup-config

[![Build Status](https://travis-ci.org/videojs/videojs-generate-rollup-config.svg?branch=master)](https://travis-ci.org/videojs/videojs-generate-rollup-config)
[![Greenkeeper badge](https://badges.greenkeeper.io/videojs/videojs-generate-rollup-config.svg)](https://greenkeeper.io/)
[![Slack Status](http://slack.videojs.com/badge.svg)](http://slack.videojs.com)

[![NPM](https://nodei.co/npm/videojs-generate-rollup-config.png?downloads=true&downloadRank=true)](https://nodei.co/npm/videojs-generate-rollup-config/)

Currently rollup configs are the same for most plugins, but when the default config changes a bit, every repository has
to be updated since it is a static file. This package will provide the standard config as a module, so that updates can be
deployed much easier.

Lead Maintainer: Brandon Casey [@brandonocasey](https://github.com/brandonocasey)

Maintenance Status: Stable


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Installation](#installation)
- [Environment variables](#environment-variables)
  - [`TEST_BUNDLE_ONLY`](#test_bundle_only)
  - [`NO_TEST_BUNDLE`](#no_test_bundle)
- [Function Options](#function-options)
  - [`input`](#input)
  - [`testInput`](#testinput)
  - [`distName`](#distname)
  - [`exportName`](#exportname)
  - [`excludeCoverage`](#excludecoverage)
  - [`coverage`](#coverage)
  - [`browserslist`](#browserslist)
  - [`checkWatch`](#checkwatch)
  - [`banner`](#banner)
  - [`babel`](#babel)
  - [`global`/`globals`](#globalglobals)
  - [`external`/`externals`](#externalexternals)
  - [`plugins`](#plugins)
  - [`primedPlugins`](#primedplugins)
- [Defaults](#defaults)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation
> Note: version 3+ support rollup ^1.0.0. If you need older rollup support please use version 2.

```
$ npm install --save-dev rollup videojs-generate-rollup-config
```

Then in your rollup config do

```js
const generateRollupConfig = require('videojs-generate-rollup-config');
const config = generateRollupConfig();

/**
 * Now you have:
 * - config.settings: An object containing the settings used to generate the builds.
 * - config.builds: An object containing the generated builds, with the name of the build as the key:
 *   - cjs
 *   - es
 *   - test
 *   - browser
 *   - minBrowser, this will be excluded by default in watch mode. See the `checkWatch` option.
 */

// Note that you have to take the values from config.builds, since rollup takes an array.
export default Object.values(config.builds);
```

Before you exports you can customize the builds, and you can also pass options (outlined below) to customize the builds.

## Environment variables
These can be passed via rollup's `--environment` or just setting an environment variable

### `TEST_BUNDLE_ONLY`
> Type: `Boolean`
> Default: false

Set this to any JavaScript truthy value, to only create a `test` build target. If this option is present `NO_TEST_BUNDLE` will be ignored.

### `NO_TEST_BUNDLE`
> Type: `Boolean`
> Default: false

Set this to any JavaScript truthy value, to create all build targets except `test`. This option will be ignored if `TEST_BUNDLE_ONLY` is set.


## Function Options
options that are passed as an object to the `generateRollupConfig` function.

### `input`

> Type: `string`
> Default: `src/plugin.js`

The entry point for your build.

### `testInput`

> Type: `string`
> Default: `test/**/*.test.js`

The entry point for tests.

### `distName`

> Type: `string`
> Default: Your package name minus scope.

Determines the dist file names for your build. IE:
`@thing/videojs-foo-bar` would have a `distName` of `videojs-foo-bar` and build to `dist/videojs-foo-var.js` etc.

### `exportName`

> Type: `string`
> Default: Your package name minus scope in camel case.

Determines the export name for browser and test dist files of your build. IE:
* `@thing/videojs-foo-bar` would have a `exportName` of `videojsFooBar`.
* The `browser`/`minBrowser` dist would be exported to window as `window.videojsFooBar`.
* The `test` dist would be exported to window as `window.videojsFooBarTests`.

### `excludeCoverage`

> Type: `Function`
> Default: no default


A callback to change the files that are excluded from test coverage.

This function takes one argument and will be passed the default files that are excluded from coverage, see below. This function **must** return the final list of files that should be excluded from coverage.

Defaults are as follows:

> NOTE: __dirname refers to the directory of `videojs-generate-rollup-config`. This is used because some rollup plugins insert
> special transforms.

```
['test/**', path.join(__dirname, '**'), 'node_modules/**', 'package.json']
```

### `coverage`

> Type: `Boolean`
> Default: true


Whether to include istanbul for the unit test bundle. By default it is included.


### `browserslist`

> Type: `string|Array`
> Default: ['defaults', 'ie 11']

What browser syntax should be supported in the browser/test dist files. Can also be specified in the package.json as a top level `browserslist` key/value. See the [browserslist repo](https://github.com/browserslist/browserslist) for more information.

### `checkWatch`

> Type: `boolean`
> Default: `true`

Determines if we should check if rollup is in watch mode. If it is we exclude the minBrowser build to save development time.

### `banner`

> Type: `string`
> Default: `/*! @name ${pkg.name} @version ${pkg.version} @license ${pkg.license} */`

The banner that should be inserted to the top of all bundles. You probably should not change this from the default!

### `babel`

> Type: `Function`
> Default: no default

A callback to change the default settings for [`rollup-plugin-babel`](https://github.com/rollup/rollup-plugin-babel).

This function takes one argument, the default babel config. This function **must** return
the final babel config that should be used for [`rollup-plugin-babel`](https://github.com/rollup/rollup-plugin-babel).

default babel config:
```js
{
  babelrc: false,
  exclude: path.join(process.cwd(), 'node_modules/**'),
  presets: [
    [presetEnv, {loose: true, modules: false, targets: {browsers: settings.browserslist}}]
  ],
  plugins: [
    transformObjectAssign
  ]
};
```

Example:

```js
const config = generateRollupConfig({
  babel(default) {
    default.plugins.push('babel-plugin-lodash');

    return default;
  }
})
```

### `global`/`globals`

> Type: `Function`
> Default: no default

A callback to change the build globals for the rollup build before they are used.

This function takes one argument and will be passed the [defaultGlobals](###defaultGlobals). This function **must** return
the final globals that should be used to create the rollup build list.

Example:

```js
const config = generateRollupConfig({
  globals(defaults) {
    return {
      browser: Object.assign(defaults.browser, {
        'foo-bar': 'foo',
      }),
      module: Object.assign(defaults.module, {
        'foo-bar': 'foo'
      }),
      test: Object.assign(defaults.test, {
        'foo-bar': 'foo'
      }),
    };
  }
})
};
```

### `external`/`externals`

> Type: `Function`
> Default: no default

A callback to change the build externals for the rollup build before they are used.

This function takes one argument and will be passed the [defaultExternals](###defaultExternals). This function **must** return
the final externals that should be used to create the rollup build list.

> NOTE: You do not have to re-list globals in this list, all globals will be added as external automatically!

Example:

```js
const config = generateRollupConfig({
  externals(defaults) {
    return {
      browser: defaults.browser.concat([
        'foo',
      ]),
      module: defaults.module.concat([
        'foo'
      ],
      test: defaults.test.concat([
        'foo'
      ],
    };
  }
});
```

### `plugins`

> Type: `Function`
> Default: no default

A callback to change the build plugins and plugin order for the rollup build before they are used.

This function takes one argument and will be passed the [defaultPlugins](###defaultPlugins). This function **must** return
the final plugins that should be used to create the rollup build list.

> NOTE: Plugins can be "primed", here or in the `primedPlugins` option. By default all plugins are primed later and
>       referenced by their key in `primedPlugins`.

Example:

```js
const config = generateRollupConfig({
  plugins(defaults) {
    return {
      browser: defaults.browser.concat([
        'foo',
      ]),
      module: defaults.module.concat([
        'foo'
      ],
      test: defaults.test.concat([
        'foo'
      ],
    };
  }
});
```

### `primedPlugins`

> Type: `Function`
> Default: no default

A callback to change the primed plugins  for the rollup build before they are used.

This function takes one argument and will be passed the [defaultPrimedPlugins](###defaultPrimedPlugins). This function **must** return
the final primed plugins that should be used to create the rollup build list.

Example:

```js
const fooPlugin = require('rollup-plugin-foo');
const config = generateRollupConfig({
  primedPlugins(defaults) {
    return Object.assign(defaults, {
      foo: fooPlugin();
    });
  }
});
```

## Defaults

See `ORDERED_DEFAULTS` in the code.

