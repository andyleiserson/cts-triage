## Test Results

The CTS test suite `webgpu:shader,validation,shader_io,pipeline_stage:*` failed to run successfully. Unfortunately, all 73 tests failed due to WebGPU device initialization issues.

### Actual Results:
- **Passed**: 0 / 73 (0.00%)
- **Failed**: 73 / 73 (100.00%)
- **Skipped**: 0 / 73 (0.00%)

### Expected Results:
- **Passed**: 67
- **Failed**: 6
- **Skipped**: 0

### Issue Analysis:
The root cause is that the WebGPU adapter initialization failed with "requestAdapter returned null". This indicates that despite using `dangerouslyDisableSandbox: true`, the Deno runtime within the test runner is unable to access the Metal/WebGPU backend on macOS.

The error message appears in the first test:
```
Error: WebGPU device failed to initialize with Error "requestAdapter returned null"; not retrying
```

This suggests that the sandbox restrictions are being applied at a different layer (possibly within the Cargo workspace or Deno runtime) that the `dangerouslyDisableSandbox` flag doesn't affect, or there may be additional system-level permissions required for WebGPU/Metal access that aren't configured.
