---
title: "Forensic container checkpointing on Amazon Elastic Kubernetes Service (Amazon EKS)"
url: "https://aws.amazon.com/blogs/containers/forensic-container-checkpointing-on-amazon-eks/"
date: "2026-08-12"
author: "Varun DeviReddy"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Amazon EKS 1.34 makes the Kubelet Checkpoint API functional, so you can capture a running container's full state (memory, processes, and network connections) without stopping the workload. This post shows how to deploy an unprivileged checkpoint agent that stores forensic checkpoints in Amazon ECR as OCI images for later analysis.
