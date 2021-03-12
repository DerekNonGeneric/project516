# project516

Test fixture for `Module.resolvePackageRoot`. See [nodejs/modules#516](https://github.com/nodejs/modules/issues/516).

## Definitions

<dl>
  <dt><strong><code>Module.resolvePackageRoot</code></strong></dt>
  <dd>Allows users to determine the <em>package root</em> of a package specifier</dd>
</dl>

<dl>
  <dt>package root</dt>
  <dd>the directory containing the <code>package.json</code> that served as the module's entrypoint</dd>
</dl>

## Syntax

```mjs
Module.resolvePackageRoot(specifier);
Module.resolvePackageRoot(specifier, base);
```

If `base` is omitted, the equivalent of [`process.cwd()`](https://nodejs.org/api/process.html#process_process_cwd) will be used.

## Examples

Omitting `base` and getting the `package.json` of a package specifier. Hypothetical minimal example.

```mjs
import module from 'module';
import path from 'path';

const tapePkgRoot = module.resolvePackageRoot('tape');
const tapePjsonPath = path.resolve(tapePkgRoot, 'package.json');
const tapePjsonData = await import(tapePjsonPath); // :p

console.log(tapePjsonData);
```

Providing `base` and getting the `package.json` of a package (bare) specifier. Example that _could_ work today.

```mjs
import { promises as fs } from 'fs';
import module from 'module';
import path from 'path';

const lodashPkgRoot = module.resolvePackageRoot('lodash', import.meta.url);
const lodashPjsonPath = path.resolve(lodashPkgRoot, 'package.json');
const lodashPjsonData = JSON.parse(await fs.readFile(lodashPjsonPath, 'utf-8')); // :/

console.log(lodashPjsonData);
```

Omitting `base` and getting the `package.json` of a package (bare) specifier in only a few lines.

```mjs
import { resolvePackageRoot } from 'module';
import { join } from 'path';

const jestPjsonData = await import(
  join(resolvePackageRoot('jest'), 'package.json'),
  { assert { type: 'json' } }); // !

console.log(jestPjsonData);
```

## Codebase layout

```tree
/app/
├── /node_modules/
│   └── /cjs-logger/
│   │   └── package.json
│   └── /esm-logger/
│   │   └── package.json
│   └── /logger/
│       ├── /cjs/
│       │   ├── logger.js
│       │   └── package.json
│       ├── /esm/
│       │   ├── logger.js
│       │   └── package.json
│       └── package.json
└── package.json
```

| /app/ | |
| --- | --- |
| `/app/node_modules/logger/package.json` | package boundary at `/app/node_modules/logger` |
| `/app/node_modules/logger/esm/package.json` | package boundary at `/app/node_modules/logger/esm` , in theory can contain `"exports"` |
| `/app/node_modules/logger/cjs/package.json` | package boundary at `/app/node_modules/logger/cjs` , in theory can contain `"exports"` |

## Further reading

- [📦 `realize-package-specifier`](https://www.npmjs.com/package/realize-package-specifier)
- [📦 `npm-package-arg`](https://www.npmjs.com/package/npm-package-arg)
