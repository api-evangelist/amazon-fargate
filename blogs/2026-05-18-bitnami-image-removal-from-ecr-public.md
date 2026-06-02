---
title: "Bitnami image removal from ECR Public"
url: "https://aws.amazon.com/blogs/containers/bitnami-image-removal-from-ecr-public/"
date: "2026-05-18"
author: "Meg Sarros"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Starting on June 10th, 2026, Bitnami container images will no longer be available on Amazon ECR Public Gallery. If you currently pull Bitnami images directly from ECR Public in your workloads, you need to take action before this date to avoid service disruption. In this post, we walk you through how to determine if you're affected, how to mirror the images you need to your own private registry, and best practices for protecting your workloads from future upstream changes.
