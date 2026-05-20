---
title: "Back up and restore your Amazon EKS cluster resources using Velero"
url: "https://aws.amazon.com/blogs/containers/back-up-and-restore-your-amazon-eks-cluster-resources-using-velero/"
date: "Tue, 12 May 2026 18:19:57 +0000"
author: "Sapeksh Madan"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
In this post, you'll learn to back up and restore Amazon EKS cluster resources and persistent volume data using Velero. You'll deploy a sample stateful application, back it up, and restore it to a different namespace within the same cluster. Along the way, you'll configure least-privilege AWS Identity and Access Management (AWS IAM) roles using Amazon EKS Pod Identity and scope Velero's Kubernetes permissions with a custom ClusterRole.
