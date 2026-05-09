---
title: "Cross-Region disaster recovery for Amazon EKS using AWS Backup"
url: "https://aws.amazon.com/blogs/containers/cross-region-disaster-recovery-for-amazon-eks-using-aws-backup/"
date: "2026-05-06"
author: "Santosh Vallurupalli"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Organizations running containerized workloads on Amazon Elastic Kubernetes Service (Amazon EKS) need resilient strategies that protect both application configurations and stateful data against Regional disruptions. While single-Region architectures with multi-AZ deployments serve many availability requirements well, some workloads demand cross-Region disaster recovery (DR) to meet stringent recovery time objectives (RTOs) and recovery point objectives (RPOs) mandated by regulatory or business continuity requirements.
