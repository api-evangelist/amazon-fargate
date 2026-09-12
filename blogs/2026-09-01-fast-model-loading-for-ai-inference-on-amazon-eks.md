---
title: "Fast model loading for AI inference on Amazon EKS"
url: "https://aws.amazon.com/blogs/containers/fast-model-loading-for-ai-inference-on-amazon-eks/"
date: "2026-09-01"
author: "Sajjan Gundapuneedi"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
When you scale AI inference on Amazon EKS, every new pod must load model weights into GPU memory before serving traffic. We investigated where cold-start time goes and found two configuration-only changes to Run:ai Model Streamer that cut model startup time by 80-93% on subsequent launches, with no code changes.
