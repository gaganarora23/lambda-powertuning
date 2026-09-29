# AWS Lambda Power Tuning - Step Function Architecture

## Overview

AWS Lambda Power Tuning uses AWS Step Functions to orchestrate a workflow that invokes the same Lambda function with different memory settings and records the results.

This is the key idea behind the tool: the same function is run repeatedly with different memory values, and the system compares execution time, latency, and cost to find the best configuration.

## Step Function workflow

```text
Start
  ↓
Initializer
  ↓
Publisher
  ↓
Choice: isCountReached?
  ├─ No → Branching → Executor → Cleaner → Analyzer → loop
  └─ Yes → CleanupOnError → End
```

## What happens in the workflow

### 1. Initializer
The workflow prepares the test environment and sets the Lambdas to be tested.

### 2. Publisher
The publisher creates or prepares the Lambda variants used by the tuning job.

### 3. Choice state: isCountReached
This step decides whether all memory options have already been tested.

### 4. Branching state
This is the important stage. The same Lambda is invoked with different memory settings, for example:

- 128 MB
- 256 MB
- 512 MB
- 768 MB
- 1024 MB

### 5. Executor
The executor calls the Lambda and measures:

- execution time
- latency
- throughput
- error rate
- cost estimate

### 6. Cleaner
This step removes temporary data and organizes the output for the analyzer.

### 7. Analyzer
The analyzer computes averages, compares runtime differences, and suggests the best memory sizes.

### 8. CleanupOnError
Final cleanup happens after all memory configurations are tested or if a failure occurs.

## Why this matches my Lambda tuning run

In my own Power Tuning run, the Lambda was effectively tested with different memory settings using the Step Functions orchestration. The important pattern is that the same function is called repeatedly while memory changes.

That is exactly what you see in the Lambda Power Tuning workflow:

- same code
- different memory values
- different runtime results
- cost and performance comparison

## My real observation

I first ran the function with the base memory configuration:

- 128 MB
- slower response
- higher latency

Then I ran the same request with:

- 512 MB
- significant improvement in execution time
- smoother and faster output

This matches the behavior that the Step Function workflow is meant to measure and compare.

## Result from my run

The observed result was:

- 128 MB had slow execution
- 512 MB was much faster
- around 75% gain in performance

This confirms the visual behavior that Lambda Power Tuning is designed to show: more memory can improve efficiency and reduce execution time for many workloads.

## Practical conclusion

The Step Function architecture is useful because it makes the trade-off visible and measurable. It turns a vague idea like “more memory is faster” into a concrete comparison across real execution results.

For my workload, the data clearly showed that 512 MB was the better configuration compared to 128 MB.

