---
title: "Amazon EKS now supports control plane egress through your VPC"
url: "https://aws.amazon.com/blogs/containers/amazon-eks-now-supports-control-plane-egress-through-your-vpc/"
date: "2026-06-22"
author: "Aaresh Sharma"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Today, we’re announcing customer-routed control plane egress, a new capability that you can use to route Kubernetes control plane traffic through your own Amazon Virtual Private Cloud (Amazon VPC). This includes admission webhook callbacks, OpenID Connect (OIDC) provider lookups, and aggregate API server requests. With this feature, you can apply the same VPC routing, security group, endpoint policy, and AWS Network Firewall controls that you use for your data plane to the Kubernetes API Server’s customer-controllable outbound traffic on Amazon Elastic Kubernetes Service (Amazon EKS) clusters.
