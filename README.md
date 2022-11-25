# rollup-plugin-webbundle

A Rollup plugin which generates
[Web Bundles](https://wicg.github.io/webpackage/draft-yasskin-wpack-bundled-exchanges.html)
output. Currently the spec is still a draft, so this package is also in alpha
until the spec stabilizes.

## Requirements

This plugin requires Node v14.0.0+ and Rollup v1.21.0+.

## Install

Using npm:

```console
npm install rollup-plugin-webbundle --save-dev
```

## Usage

### General Web Bundle

This example assumes your application entry point is `src/index.js` and static
files (including `index.html`) are located in `static` directory.

```js
/* rollup.config.mjs */
import webbundle from 'rollup-plugin-webbundle';

export default {
  input: 'src/index.js',
  output: {
    dir: 'dist',
    format: 'esm',
  },
  plugins: [
    webbundle({
      baseURL: 'https://example.com/',
      static: { dir: 'static' },
    }),
  ],
};
```

A WBN file `dist/out.wbn` should be written.

### Isolated Web App (Signed Web Bundle)

This example assumes your application entry point is `src/index.js`, static
files (including `index.html`) are located in `static` directory and you have a
`.env` file in the root directory with `ED25519KEY` defined in it. The example
also requires installing `dotenv` and `wbn-sign` npm packages as dev
dependencies.

```js
/* rollup.config.mjs */
import webbundle from 'rollup-plugin-webbundle';
import * as wbnSign from 'wbn-sign';
import dotenv from 'dotenv';
dotenv.config({ path: './.env' });

export default {
  input: 'src/index.js',
  output: {
    dir: 'dist',
    format: 'esm',
  },
  plugins: [
    webbundle({
      baseURL: new wbnSign.WebBundleId(
        wbnSign.parsePemKey(process.env.ED25519KEY)
      ).serializeWithIsolatedWebAppOrigin(),
      static: { dir: 'public' },
      output: 'signed.swbn',
      integrityBlockSign: {
        key: process.env.ED25519KEY,
      },
    }),
  ],
};
```

A web bundle with integrity block (.swbn) `dist/signed.swbn` should be written.

## Options

### `baseURL` (required)

Type: `string`

Specifies the URL prefix prepended to the file names in the bundle. This must be
an absolute URL that ends with `/`.

### `formatVersion`

Type: `string`  
Default: `b2`

Specifies WebBundle format version.

### `primaryURL`

Type: `string`  
Default: baseURL

Specifies the bundle's main resource URL. If omitted, the value of the `baseURL`
option is used.

### `static`

Type: `{ dir: String, baseURL?: string }`

If specified, files and subdirectories under `dir` will be added to the bundle.
`baseURL` can be omitted and defaults to `Options.baseURL`.

### `output`

Type: `string`  
Default: `out.wbn`

Specifies the file name of the Web Bundle to emit.

### `integrityBlockSign`

Type: `{ key: string }`

Object specifying the signing options with Integrity Block.

### `integrityBlockSign.key` (required if `integrityBlockSign` in place)

Type: `string`

Ed25519 type of private key as a string, which can be generated with:

```bash
openssl genpkey -algorithm Ed25519 -out ed25519key.pem
```

Note than in order for it to be parsed correctly, it must contain the `BEGIN`
and `END` texts and line breaks (`\n`). Below an example `.env` file:

```bash
ED25519KEY="-----BEGIN PRIVATE KEY-----\nMC4CAQAwBQYDK2VwBCIEIB8nP5PpWU7HiILHSfh5PYzb5GAcIfHZ+bw6tcd/LZXh\n-----END PRIVATE KEY-----"
```

## License

Licensed under the Apache-2.0 license.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) file.

## Disclaimer

This is not an officially supported Google product.

## Release Notes

### v0.0.4

- Support for signing web bundles with
  [integrity block](https://github.com/WICG/webpackage/blob/main/explainers/integrity-signature.md)
  added.
