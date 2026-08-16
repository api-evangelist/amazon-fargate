---
title: "Centralize cross-account Amazon ECS telemetry with an ADOT gateway"
url: "https://aws.amazon.com/blogs/containers/centralize-cross-account-amazon-ecs-telemetry-with-an-adot-gateway/"
date: "2026-08-06"
author: "Rahul Kumar"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Running an OpenTelemetry collector as a sidecar in every Amazon ECS task does not scale across a multi-account estate, and it cannot run at all on Windows. Learn how to replace per-task sidecars with a single centralized ADOT gateway that ingests OTLP from workloads across accounts and exports traces to AWS X-Ray and metrics and logs to Amazon CloudWatch.
