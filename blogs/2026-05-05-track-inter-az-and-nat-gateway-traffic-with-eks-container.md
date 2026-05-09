---
title: "Track inter-AZ and NAT gateway traffic with EKS Container Network Observability"
url: "https://aws.amazon.com/blogs/containers/track-inter-az-and-nat-gateway-traffic-with-eks-container-network-observability/"
date: "2026-05-05"
author: "Arik Porat"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Teams running microservices on Amazon Elastic Kubernetes Service (Amazon EKS) struggle to identify which services drive their data transfer costs. As clusters grow and service-to-service communication increases, inter-AZ traffic (data transfer between zones within the same region) and NAT gateway processing charges add up fast. Without pod-level visibility into these network flows, it’s difficult to pinpoint which workloads contribute most.
