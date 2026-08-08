---
title: "Run GPU batch inference on Amazon ECS Managed Instances with scale to zero"
url: "https://aws.amazon.com/blogs/containers/run-gpu-batch-inference-on-amazon-ecs-managed-instances-with-scale-to-zero/"
date: "2026-08-03"
author: "Henrique Santana"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Deploy a single CloudFormation stack that builds a GPU batch inference pipeline on Amazon ECS Managed Instances. It uses Amazon SQS for job buffering and Application Auto Scaling to scale to zero when idle, so you pay only for active inference time.
