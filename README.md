Documentation of an issue arising from pnpm-style node_modules and esbuild externals.

Reproduction:
```sh
pnpm install
pnpm build    # esbuild index.js --bundle --external:is-odd --outfile=out.js
pnpm run      # node out.js
```

Expect to see an error like:
```
node:internal/modules/cjs/loader:1147
  throw err;
  ^

Error: Cannot find module 'is-odd'
Require stack:
- /Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transitive-external/out.js
    at Module._resolveFilename (node:internal/modules/cjs/loader:1144:15)
    at Module._load (node:internal/modules/cjs/loader:985:27)
    at Module.require (node:internal/modules/cjs/loader:1235:19)
    at require (node:internal/modules/helpers:176:18)
    at node_modules/.pnpm/is-even@1.0.0/node_modules/is-even/index.js (/Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transit
ive-external/out.js:39:19)
    at __require2 (/Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transitive-external/out.js:16:52)
    at /Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transitive-external/out.js:47:32
    at Object.<anonymous> (/Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transitive-external/out.js:49:3)
    at Module._compile (node:internal/modules/cjs/loader:1376:14)
    at Module._extensions..js (node:internal/modules/cjs/loader:1435:10) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/Users/ryan/src/github.com/ryanwilsonperkin/esbuild-pnpm-transitive-external/out.js'
  ]
}

Node.js v20.11.1
```

## Explanation

The JS file imports a package `is-even` which has a transitive dependency on
`is-odd`. When building, we mark `is-odd` as external. Under a classic style
node_modules hierarchy, `is-odd` would have been hoisted up to the top-level
under `node_modules/` and would be discovered by Node's module resolution
algorithm. Under pnpm, this package is instead found under
`node_modules/is-even/node_modules/is-odd` (via a series of symlinks).

Since `is-even` is bundled, its relative path is not applied as part of
`module.paths` when evaluating the import of `is-odd`, so it is unable to find
it at this nested path.
