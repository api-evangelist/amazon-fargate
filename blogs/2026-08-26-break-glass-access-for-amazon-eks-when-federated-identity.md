---
title: "Break-glass access for Amazon EKS when federated identity fails"
url: "https://aws.amazon.com/blogs/containers/break-glass-access-for-amazon-eks-when-federated-identity-fails/"
date: "2026-08-26"
author: "Sam Mukherjee"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Implementing break-glass access for Amazon EKS clusters removes the circular dependency where a federated identity provider outage locks you out of the clusters you need to reach to fix it. This post supplies a cross-account IAM role with enforced MFA, infrastructure-as-code templates, validation tests, and a post-incident recovery procedure.
