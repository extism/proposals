#  Integration test framework

## Purpose

Extism have many language's SDk and PDK, which is excellent! but we're lack of tests to make sure all SDK and PDK are working expectedly.

for example: we made a change to GO-PDK, tested with GO-SDK, Rust-SDK but breaks JS-SDK.

So we need a integration test framework to test over all SDK/PDK permutations.

## Solution

### Key concepts

#### TestTarget
Represents the test running environment,  
Includes env(different toolchains), PDK(and/or version), SDK(and/or version).

example:
```yaml
env:
  - tiny-go=23.0.1
sdk: js-sdk=2.0.0-rc
pdk: go-pdk=latest
```

env feature is optional.

#### TestSuite
Represents a group of TestCases with options.

options:
  - targets: target matchers, use to skip unimplemented/unsupported target or env

#### TestCase
- inside a TestSuite
- have an expected output to validate


### Workflow

1. design TestSuite and TestCases for ALL SDK/PDK
2. implement TestSuites
    - each PDK/SDK to test should implement the TestSuite code, follow the expected output
    - store in a standardlong repo?
3. define TestTargets(optional)
    - can define additaional TestTargets to run test suites
4. trigger runner by new version released or by manual


### Runflow
Use your favorite build tools to implement, recomment base on Docker.  

1. scan TestSuite and TestCases to generate inital TestTargets
2. mix inital TestTargets with additaional TestTargets
3. prepare environment
    - compile wasm binaries with specified toolchain and PDK
    - prepare SDKs
4. run all TestTargets
5. check output is expected and generate test report


## Considerations
- need a Runner to run SDK code and PDK binary, also can be used to benchmark
- better not use assert library to check output, just compare output itself to keep it simple.
- TestSuite versioning, consider SDK/PDK may introduce breaking change
- prevent overcomplicated
