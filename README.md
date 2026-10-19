# vize-ssr-arrow-param-repro

Minimal reproduction for a false positive in [vize](https://github.com/ubugeeei-prod/vize)'s `ssr/no-browser-globals-in-ssr`.

In an inline handler, a parameter named like a browser global (`open`, as in `window.open`) is reported as that global, once per occurrence. The parameter shadows the global.

## Steps

```sh
pnpm install
pnpm lint
```

[`src/ArrowParam.vue`](./src/ArrowParam.vue):

```vue
<template>
  <button @click="(open) => open">Open</button>
</template>
```

## Expected

No warning.

## Actual

```
Patina lint report: 2 warnings in 1 file

src/ArrowParam.vue
  src/ArrowParam.vue:2:11 warning ssr/no-browser-globals-in-ssr 'open' is a browser-only global and is not available in SSR
    Reference: docs/content/rules/ssr.md
  src/ArrowParam.vue:2:11 warning ssr/no-browser-globals-in-ssr 'open' is a browser-only global and is not available in SSR
    Reference: docs/content/rules/ssr.md
```

## Notes

- A `function (open) { return open }` handler is reported the same way, and so are other browser-global names such as `location`.
- `v-for="open in …"` and slot props named `open` are not reported.
- [`extract_identifiers`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/ssr/no_browser_globals_in_ssr/template_identifiers.rs#L12) scans the expression without knowing function parameters.
- [`is_local_binding`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/ssr/no_browser_globals_in_ssr.rs#L195-L203) accepts v-for variables, script bindings and other analysed template bindings, but an inline handler's parameters are not among them.

## Environment

vize 0.476.0, Node v26.10.0, pnpm 12.9.0, macOS (arm64)
