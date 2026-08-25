---
title: "Deep dive into Amazon EKS certificate authority rotation"
url: "https://aws.amazon.com/blogs/containers/deep-dive-into-amazon-eks-certificate-authority-rotation/"
date: "2026-08-19"
author: "Micah Hausler"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
Amazon EKS now provides a managed, non-disruptive lifecycle for rotating your cluster's certificate authority (CA), with automated safeguards and rollback. This deep dive explains how CA rotation works, what AWS handles versus what you must update, and how to walk through the rotation lifecycle on your own timeline.
