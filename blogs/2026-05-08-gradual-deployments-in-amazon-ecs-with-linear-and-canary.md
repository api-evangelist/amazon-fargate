---
title: "Gradual deployments in Amazon ECS with linear and canary strategies"
url: "https://aws.amazon.com/blogs/containers/gradual-deployments-in-amazon-ecs-with-linear-and-canary-strategies/"
date: "2026-05-08"
author: "Rishabh Dubey"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
When deploying new application versions, you need confidence changes won’t impact customers. Amazon Elastic Container Service (Amazon ECS) now supports linear and canary deployment strategies, complementing built-in blue/green deployments. With linear deployments, you shift traffic in equal increments with a bake time between each shift. With canary deployments, you route a small percentage to the new revision and monitor before shifting the rest. Both strategies support Amazon CloudWatch alarms for failure detection and rollback, and lifecycle hooks for custom validation.
