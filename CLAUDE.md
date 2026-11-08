# flashpay

`@liteimport/flashpay`: a TypeScript client for the FlashPay payment gateway, for applications that
take Thai PromptPay QR and mobile banking payments and send bank payouts. The API, parameters,
status codes and bank codes are in [README.md](README.md).

All the code is in `src/client.ts`; `src/index.ts` names the public exports.

## Commands

```sh
npm ci           # also runs prepare (tsc)
npm run build    # tsc into dist/
```

These were run on 2026-11-08 under Node 22 and succeeded; the build left `dist/` unchanged. There
are no tests and no `test` script, and the repository has no CI workflows. To check a change, build
it and exercise it with a stubbed `fetch`. Nothing described here was run against the gateway.

## Invariants

- **Web Crypto only.** Signing uses `crypto.subtle`, `atob` and `btoa` rather than `node:crypto`, so
  that the package works in an Edge runtime. Do not add `node:` imports or `Buffer`.
- **Signature format.** The string to sign is `key=value` pairs sorted by key and joined with `&`.
  `data` is serialised as JSON with its keys sorted, after dropping `null`, `undefined` and empty
  string values. `sign` and `signType` are added after signing. The gateway checks this string, so
  any change to `createDataForSignature` breaks every request.
- **`dist/` is committed**, for Git installs. Rebuild it in the same commit as any change to `src/`.
  `prepare` runs `tsc` on install, which is why `typescript` is a runtime dependency.
- **Module format.** `tsconfig.json` uses `NodeNext` and `package.json` has no `type` field, so
  `dist/` is CommonJS. Relative imports in `src/` must keep their `.js` extension.
- **Amounts** are integers in the smallest currency unit: `10000` is 100.00 THB.

## Traps

These were confirmed by running the built package with a stubbed `fetch` and generated keys, except
where the text says otherwise.

- **Key format.** Private keys are imported as PKCS#8 and public keys as SPKI. A PEM key in PKCS#1
  format (`BEGIN RSA PRIVATE KEY`) fails with `Invalid keyData`. The README mentions PEM but not
  this restriction.
- **`success` is only `tradeStatus === 3`.** `handleNotification` returns `success: false` for every
  other status, including status 2 (processing), so `false` does not mean the payment failed. It
  also does not compare the amount with the order; the caller must.
- **Timestamps** (`time`, `outTradeTime`, the default `expireTime`) are formatted by `date-fns` in the
  time zone of the process. This one is read from the code, not run.
- **Installation.** `npm view @liteimport/flashpay` returned 404 on 2026-11-08, although the README
  shows `pnpm add @liteimport/flashpay`. Under pnpm 10, `pnpm add github:szyfkj/flashpay` fails with
  `ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED` unless the package is in `onlyBuiltDependencies`. With that
  setting it installed and loaded.

## How changes land

The base branch is `main`, which requires a pull request. There are no version tags.
