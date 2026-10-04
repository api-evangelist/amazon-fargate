---
title: "Fix pod distribution drift in Amazon EKS with the Kubernetes descheduler"
url: "https://aws.amazon.com/blogs/containers/fix-pod-distribution-drift-in-amazon-eks-with-the-kubernetes-descheduler/"
date: "2026-09-28"
author: "Ramya D"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
A workload spread across three Availability Zones does not necessarily stay spread. This post explains why soft topology spread constraints drift after a node-availability gap, measures the cost, and shows how the Kubernetes descheduler restores even pod distribution on Amazon EKS without downtime and without forcing hard constraints.
