# buffer,create CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,buffer,create:*`

Model: Opus

Platform: macOS / Metal

**Overall Status:** 84P/1F/0S (98.8%/1.2%/0%)

## Passing Sub-suites ✅

- `createBuffer_invalid_and_oom` (1 test) — mappable buffers with an invalid
  usage combination (`MAP_*` plus `UNIFORM`) correctly report a validation
  error rather than an OOM error, including at `kMaxSafeMultipleOf8` and
  128 GiB sizes.
- `limit` (3 tests) — `size` at `maxBufferSize - 1`, `maxBufferSize`, and
  `maxBufferSize + 1`.
- `size` (2 tests) — size alignment/`mappedAtCreation` rules.
- `usage` (78 tests) — every `usage1 | usage2` pair drawn from
  `kBufferUsages`, with `mappedAtCreation` both false and true.

## Remaining Issues ⚠️

- `new_usages` (1 test, 10 subcases) — all 10 subcases fail with
  `EXPECTATION FAILED: Expected validation error`. Root cause is in the
  deno_webgpu JavaScript bindings, not in wgpu-core validation.

## Issue Detail

### 1. `new_usages` — `GPUBufferUsage` constants are not enumerable

**Test selector:** `webgpu:api,validation,buffer,create:new_usages:`

**What it tests:** That usage bits which the implementation does *not* expose
on the `GPUBufferUsage` interface object are rejected by `createBuffer()`. The
test derives the set of legal bits from the implementation itself:

```js
let exposedUsages = 0;
for (const v of Object.values(GPUBufferUsage)) {
  exposedUsages |= v;
}
const success = (usage & exposedUsages) === usage;
t.expectGPUError('validation', () => { t.createBufferTracked({ size: 16, usage }); }, !success);
```

**Example failure:**

```
webgpu:api,validation,buffer,create:new_usages:
```

**Error:** every subcase `usage=1`, `2`, `4`, `8`, `16`, `32`, `64`, `128`,
`256`, `512` reports

```
EXPECTATION FAILED: Expected validation error
    at AllFeaturesMaxLimitsGPUTest.expectGPUError (webgpu/gpu_test.js:1163:10)
    at RunCaseSpecific.fn (webgpu/api/validation/buffer/create.spec.js:104:5)
```

**Root cause:**
In `deno_webgpu/01_webgpu.js` (lines 390-425) `GPUBufferUsage` is declared as
an ES class whose constants are **static getters**:

```js
class GPUBufferUsage {
  constructor() { webidl.illegalConstructor(); }
  static get MAP_READ() { return 0x0001; }
  static get MAP_WRITE() { return 0x0002; }
  // ... through QUERY_RESOLVE
}
```

ECMAScript class accessor properties are created with `enumerable: false`
(verified: `Object.getOwnPropertyDescriptor` on such a property yields
`{enumerable: false, configurable: true}`). WebIDL requires interface
constants to be own properties with
`{writable: false, enumerable: true, configurable: false}`.

Consequently `Object.values(GPUBufferUsage)` returns `[]`, `exposedUsages`
evaluates to `0`, and `success` is `false` for every non-zero usage. The test
therefore demands a validation error for all ten perfectly legal buffer usage
bits. wgpu correctly creates the buffers, so all ten subcases fail.

This is not a wgpu validation gap: with a spec-conformant
`GPUBufferUsage`, `exposedUsages` would be `0x3FF`, `success` would be true
for every subcase, and the test would pass unchanged (the creations already
succeed today).

Tracked as https://github.com/denoland/deno/issues/33846.

**Fix needed:**
Define the `GPUBufferUsage` (and `GPUTextureUsage`, `GPUShaderStage`,
`GPUMapMode`, `GPUColorWrite`) constants as enumerable, non-writable,
non-configurable own data properties on the interface object, e.g. via
`ObjectDefineProperties` with `enumerable: true`, instead of static getters.

## Notes

The whole file is already accounted for in the expectation lists, and the
measured results agree with them:

- `cts_runner/test.lst` lists `createBuffer_invalid_and_oom,*`, `limit,*`,
  `size,*` and `usage,*` — 1 + 3 + 2 + 78 = 84 cases, all passing.
- `cts_runner/fail.lst` lists `new_usages,*` — 1 case, failing.

Note that `cargo xtask cts verify-selectors` reports these 85 cases as
unmatched. That is a false positive: `Pattern::matches` in
`xtask/src/cts/verify_selectors.rs` is a literal glob, so a trailing
path-level `,*` wildcard does not match a listed selector that continues with
`:`, even though CTS itself treats `test,*` as matching the bare `test` and
all of its cases.
