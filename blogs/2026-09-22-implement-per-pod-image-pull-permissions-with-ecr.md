---
title: "Implement per-pod image pull permissions with ECR repository policies on Amazon EKS"
url: "https://aws.amazon.com/blogs/containers/implement-per-pod-image-pull-permissions-with-ecr-repository-policies-on-amazon-eks/"
date: "2026-09-22"
author: "Asiel Bencomo Corona"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Learn how to scope Amazon ECR image pull permissions to individual Kubernetes pods on a multi-tenant Amazon EKS cluster using KEP 4412 credential providers and ECR repository deny policies, so teams sharing the same nodes can pull only their own container images.
