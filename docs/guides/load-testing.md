# Load Testing

Load Testing is a process of ensuring your API performs as expected during a typical and peak load without breaking or becoming unresponsive

The goal of API load testing is typically to identify potential bottlenecks or issues that may arise when the API is used at scale, and to provide information that can be used to optimize the API for better performance

::: warning
This feature is currently in preview. Some things may break or change in the future
:::

1. Create a workflow

    **loadtest.yml**

    ```yaml
    version: "1.1"
    name: Load Test
    env:
      host: example.com
    tests:
      example:
        steps:
          - name: GET request
            http:
              url: https://${{env.host}}
              method: GET
    ```

2. Add the a load test configuration

    **loadtest.yml**

    ```yaml{5-11}
    version: "1.1"
    name: Load Test
    env:
      host: example.com
    config:
      loadTest:
        phases:
          - duration: 2
            arrivalRate: 1
          - duration: 10
            arrivalRate: 2
    tests:
      example:
        steps:
          - name: GET request
            http:
              url: https://${{env.host}}
              method: GET
    ```

    In the following example, we configure a load test for a http service. Load-testing gRPC APIs is also possible

    Your load test can have multiple phases. Each phase has a `duration`, which defines how long a phase lasts and an `arrivalRate`, which sets how many virtual visitors (or requests) arrive per second during the duration of a phase

    :::warning
    By default, the test runner will skip subsequent steps after the previous ones have failed. Add `continueOnFail` configuration option to your workflow or your test if you want to continue test execution after a failed step
    :::

3. Add (optional) checks

    **loadtest.yml**

    ```yaml{12-14}
    version: "1.1"
    name: Load Test
    env:
      host: example.com
    config:
      loadTest:
        phases:
          - duration: 2
            arrivalRate: 1
          - duration: 10
            arrivalRate: 2
        check:
          p99:
            - lte: 500
    tests:
      example:
        steps:
          - name: GET request
            http:
              url: https://${{env.host}}
              method: GET
    ```

    Checks are optional and allow you to validate that the resulted performance metrics match your expected values

    :::info
    See [Workflow Syntax](/reference/workflow-syntax#config-loadtest-check) for available load test checks
    :::

3. Run the load test

    ```
    stepci run --loadtest loadtest.yml
    ```

    Result:

    ```
    response_time:
      min: ............................................................ 437
      max: ............................................................ 500
      avg: ............................................................ 453.3636363636364
      med: ............................................................ 446.5
      p95: ............................................................ 498
      p99: ............................................................ 500
    steps:
      failed: ......................................................... 0
      passed: ......................................................... 22
      skipped: ........................................................ 0
      errored: ........................................................ 0
      total: .......................................................... 22
    tests:
      failed: ......................................................... 0
      passed: ......................................................... 22
      total: .......................................................... 22
    checks
      p99: ............................................................ pass
    rps: .............................................................. 1.5705311250713878
    iterations: ....................................................... 22
    duration: ......................................................... 14008

    Workflow passed after 14.008s
    ```
