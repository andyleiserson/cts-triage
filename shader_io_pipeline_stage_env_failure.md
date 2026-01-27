## Test Results

I ran the CTS test suite `webgpu:shader,validation,shader_io,pipeline_stage:*` and here are the exact results:

### Actual Results:
- **Passed**: 0 / 73 = 0.00%
- **Skipped**: 0 / 73 = 0.00%
- **Failed**: 73 / 73 = 100.00%

### Expected Results:
- **Passed**: 67
- **Failed**: 6

### Comparison:
The actual results are **significantly different** from the expected results. Instead of having 67 passing tests and 6 failures, **all 73 tests failed** (100% failure rate).

### Root Cause:
All tests are failing with the same error: **"WebGPU device failed to initialize with Error 'requestAdapter returned null'"**. This indicates that the WebGPU adapter is not available or cannot be initialized, which is causing every single test to fail before it can even run the actual test logic.

This is a complete failure of the test environment rather than failures in the specific shader validation tests themselves. The WebGPU device initialization is failing, which prevents any tests from executing properly.
