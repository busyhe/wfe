# Installation
> `npm install --save @types/babel__standalone`

# Summary
This package contains type definitions for @babel/standalone (https://github.com/babel/babel/tree/master/packages/babel-standalone).

# Details
Files were exported from https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/babel__standalone.
## [index.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/babel__standalone/index.d.ts)
````ts
// Type definitions for @babel/standalone 7.1
// Project: https://github.com/babel/babel/tree/master/packages/babel-standalone
// Definitions by: DefinitelyTyped <https://github.com/DefinitelyTyped>
// Definitions: https://github.com/DefinitelyTyped/DefinitelyTyped
// Minimum TypeScript Version: 3.4

import { TransformOptions, types, FileResultCallback, BabelFileResult } from '@babel/core';

export function transform(code: string, options: TransformOptions): BabelFileResult;

export function transformFromAst(ast: types.Node, code: string | undefined, opts: TransformOptions | undefined, callback?: FileResultCallback): void;

export function registerPlugin(name: string, plugin: object | (() => void)): void;

export function registerPlugins(newPlugins: {
    [key: string]: object | (() => void)
}): void;

export function registerPreset(name: string, preset: object | (() => void)): void;
export function registerPresets(newPresets: {
    [key: string]: object | (() => void),
}): void;

export const availablePlugins: Record<string, object | (() => void)>;
export const availablePresets: Record<string, object | (() => void)>;

export function transformScriptTags(scriptTags?: HTMLCollection): void;

export function disableScriptTags(): void;

export as namespace babel;

````

### Additional Details
 * Last updated: Thu, 23 Dec 2021 23:34:18 GMT
 * Dependencies: [@types/babel__core](https://npmjs.com/package/@types/babel__core)
 * Global values: `babel`

# Credits
These definitions were written by [DefinitelyTyped](https://github.com/DefinitelyTyped).
