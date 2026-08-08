---
title: "Under the hood: how Amazon EKS Auto Mode detects, repairs, and diagnoses node failures"
url: "https://aws.amazon.com/blogs/containers/under-the-hood-how-amazon-eks-auto-mode-detects-repairs-and-diagnoses-node-failures/"
date: "2026-08-05"
author: "Sajjan Gundapuneedi"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
On Amazon EKS Auto Mode, node failures are detected, drained, and replaced automatically before anyone reaches for a laptop. This post shows how the Node Monitoring Agent and Karpenter form a detect-and-replace cycle that runs by default, why specific faults trigger node replacement, and how to collect node diagnostics without SSH.
