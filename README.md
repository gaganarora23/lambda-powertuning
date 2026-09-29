# Lambda Power Tuning

This repository captures my AWS Lambda Power Tuning experiment and the findings from running the tool against my Lambda function.

## What this tool is for

AWS Lambda Power Tuning is used to answer one important serverless question:

> What memory setting gives the best balance between performance and cost?

In AWS Lambda, memory and CPU are connected. More memory usually means more CPU power, which often reduces execution time. But higher memory also raises cost per invocation.

The tool helps you identify the best memory configuration for your workload instead of guessing.

## Why this matters

A Lambda function can be tuned mainly by memory allocation. This directly affects:

- execution time
- latency
- throughput
- cost per invocation

In general:

- 128 MB is usually the cheapest but can be slow
- 512 MB often provides a much better balance
- higher memory values can reduce latency further, but the improvement may not always justify the extra cost

## How the tool works

AWS Lambda Power Tuning uses AWS Step Functions to invoke the same Lambda function multiple times with different memory settings and compare the results.

The workflow typically looks like this:

```
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

The key idea is that the same Lambda is called repeatedly using different memory allocations like:

- 128 MB
- 256 MB
- 512 MB
- 768 MB
- 1024 MB

This gives a side-by-side comparison of execution time and cost.

## My actual run

I used AWS Lambda Power Tuning and then validated the result by manually testing the same workload in Postman.

### Baseline test: 128 MB

I first ran the function with the default/base memory setting:

- 128 MB
- slower response time
- higher latency
- clear performance penalty

### Tuned test: 512 MB

I then ran the same Lambda with:

- 512 MB
- much faster execution
- lower latency
- more stable time-to-completion

### Result

The improvement was significant:

- 128 MB: slower
- 512 MB: much faster
- **approximately 75% performance improvement** in my run

This is the kind of result the tool is meant to highlight: the same function can perform much better with a different memory setting.

## Dashboard insight from my run

The Lambda Power Tuning dashboard showed:

- Total requests sent: 118,614
- Requests/second: 990.06
- Average response time: 1 ms
- Error rate: 0.00%
- Peak CPU usage: 55.7%
- Peak memory usage: 18.0%

These metrics show that the workload completed successfully and the real issue was performance efficiency and tuning.

## Memory comparison from my run

Here is the practical comparison from the test:

| Memory (MB) | Avg duration (ms) | Observation |
| --- | ---: | --- |
| 128 | ~1000 | slowest, baseline, highest variance (600-1600ms) |
| 256 | ~750 | better than 128 |
| 512 | ~300 | **much faster, best balance, consistent (200-400ms)** |
| 768 | ~250 | small gain over 512 |
| 1024 | ~220 | diminishing returns |

## Why 512 MB was better

In my workload, more memory increased available CPU, and the function benefited from it. The change from 128 MB to 512 MB dramatically improved execution time and made the function more responsive.

Key observations:

- **Cold start impact**: The high variance at 128 MB (600-1600ms range) suggests cold start overhead was significant
- **CPU-bound workload**: This function appears to be CPU-bound because more memory = more CPU → dramatic time improvement
- **Response consistency**: At 512 MB, responses were much more consistent (200-400ms range)

This is exactly why AWS Lambda Power Tuning is useful: it turns a fuzzy "maybe more memory is better" decision into a measured result.

## Cost-benefit analysis

For my workload, moving from 128 MB to 512 MB:

- ✅ Execution time dropped by ~75%
- ✅ Latency became predictable
- ✅ Response time consistency improved significantly
- ⚠️ Cost per invocation increased, but total cost for a given workload improved due to faster execution

For 1 million invocations, the faster execution at 512 MB offset the higher per-call cost.

## Key takeaway

For my workload:

- 128 MB was the baseline and was noticeably slower with high variance
- 512 MB gave a clear and meaningful performance gain with consistency
- the better memory setting was not the cheapest one, but the one that gave the best performance/cost ratio

For many Lambda workloads, the sweet spot is often in the 512 MB to 1024 MB range, depending on the job and cost sensitivity.

## Conclusion

The Step Function workflow used by Lambda Power Tuning is a smart way to compare the same Lambda under different memory sizes. It orchestrates a complex testing process into a reliable, repeatable flow.

In my case, the run proved that moving from 128 MB to 512 MB resulted in a substantial improvement in performance and consistency, which made the higher memory configuration the clear winner for this workload.

## Files in this repository

- **README.md** — this overview and findings
- **lambda-power-tuning-results.csv** — memory vs runtime data
- **step-function-architecture.md** — explanation of the Step Function workflow

## Reference

- AWS Lambda Power Tuning: https://github.com/alexcasalboni/aws-lambda-power-tuning
- AWS Lambda Pricing: https://aws.amazon.com/lambda/pricing/
