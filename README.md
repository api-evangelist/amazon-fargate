# Amazon Fargate (amazon-fargate)

Amazon Fargate is a serverless compute engine for containers that works with both Amazon ECS and Amazon EKS. Fargate removes the need to provision and manage servers, letting you specify and pay for resources per application, and improves security through application isolation by design.

**URL:** [https://aws.amazon.com/fargate/](https://aws.amazon.com/fargate/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Compute, Containers, ECS, EKS, Microservices, Serverless

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-04-19

## APIs

### Amazon Fargate API
The Amazon Fargate API is accessed through Amazon ECS and enables you to run containers without managing servers or clusters. You can define tasks, configure networking and IAM policies, and deploy containerized applications with serverless compute capacity.

**Human URL:** [https://aws.amazon.com/fargate/](https://aws.amazon.com/fargate/)

#### Tags:

 - Compute, Containers, Microservices, Serverless

#### Properties

- [Documentation](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [OpenAPI](openapi/amazon-fargate-openapi.yml)
- [JSONSchema](json-schema/amazon-fargate-cluster-schema.json)
- [JSONSchema](json-schema/amazon-fargate-task-definition-schema.json)
- [JSONSchema](json-schema/amazon-fargate-task-schema.json)
- [JSONSchema](json-schema/amazon-fargate-service-schema.json)
- [JSONStructure](json-structure/amazon-fargate-cluster-structure.json)
- [JSONStructure](json-structure/amazon-fargate-task-definition-structure.json)
- [JSONStructure](json-structure/amazon-fargate-task-structure.json)
- [JSONStructure](json-structure/amazon-fargate-service-structure.json)
- [Example](examples/amazon-fargate-cluster-example.json)
- [Example](examples/amazon-fargate-task-definition-example.json)
- [Example](examples/amazon-fargate-task-example.json)
- [Example](examples/amazon-fargate-service-example.json)
- [Pricing](https://aws.amazon.com/fargate/pricing/)
- [GettingStarted](https://aws.amazon.com/fargate/getting-started/)
- [FAQ](https://aws.amazon.com/fargate/faqs/)
- [APIReference](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/Welcome.html)

## Common Properties

- [Portal](https://console.aws.amazon.com/)
- [Website](https://aws.amazon.com/fargate/)
- [Documentation](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/containers/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/ecs)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [StatusPage](https://status.aws.amazon.com/)
- [YouTube](https://www.youtube.com/user/AmazonWebServices)
- [StackOverflow](https://stackoverflow.com/questions/tagged/aws-fargate)
- [SpectralRules](rules/amazon-fargate-spectral-rules.yml)
- [NaftikoCapability](capabilities/shared/fargate.yaml)
- [NaftikoCapability](capabilities/amazon-fargate-container-orchestration.yaml)
- [Vocabulary](vocabulary/amazon-fargate-vocabulary.yaml)
- [JSON-LD](json-ld/amazon-fargate-context.jsonld)

## Features

| Name | Description |
|------|-------------|
| Serverless Compute | Run containers without provisioning or managing servers. Fargate handles capacity, OS updates, and scaling automatically. |
| ECS and EKS Integration | Works seamlessly with both Amazon ECS task definitions and Amazon EKS pods. |
| Workload Isolation | Each task runs in its own dedicated single-tenant compute environment for improved security. |
| VPC Networking | Tasks receive ENIs with full VPC networking support including security groups and VPC Flow Logs. |
| Auto Scaling | Supports Application Auto Scaling with target tracking, step scaling, and scheduled scaling. |
| Persistent Storage | Integration with Amazon EFS for stateful workloads requiring persistent storage. |
| Compliance Support | HIPAA, PCI, FedRAMP, and GovCloud (US) region support for regulated workloads. |
| CloudWatch Integration | Built-in Container Insights for metrics, logs, and observability. |
| ARM64/Graviton Support | Run workloads on AWS Graviton processors for improved price-performance. |
| Spot Instances | Run fault-tolerant workloads on Fargate Spot for significant cost savings. |

## Use Cases

| Name | Description |
|------|-------------|
| Web Applications and APIs | Deploy microservices-based web applications and REST APIs without infrastructure management. |
| Batch Data Processing | Run parallel data processing jobs and ETL workloads using AWS Batch with Fargate. |
| Application Modernization | Lift-and-shift containerized workloads to serverless infrastructure for reduced operational burden. |
| AI/ML Workloads | Run training, inference, and data preparation containers in flexible serverless environments. |
| CI/CD Pipelines | Execute build, test, and deployment pipelines as ephemeral Fargate tasks. |
| Scheduled Jobs | Run time-based container workloads using Amazon EventBridge and Fargate tasks. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon ECS | Primary orchestration engine for running Fargate tasks and services. |
| Amazon EKS | Run Kubernetes pods serverlessly using Fargate profiles. |
| AWS IAM | Fine-grained task-level IAM roles for container security and least privilege. |
| Amazon CloudWatch | Container Insights, metrics, logs, and alarms for Fargate workloads. |
| AWS Application Auto Scaling | Automatically scale Fargate services based on CloudWatch metrics. |
| Amazon EFS | Persistent shared file storage for stateful Fargate workloads. |
| AWS Batch | Run high-scale batch workloads using Fargate compute environments. |
| Application Load Balancer | Route HTTP/HTTPS traffic to Fargate services using ALB target groups. |
| AWS Secrets Manager | Inject secrets and configuration into Fargate task containers securely. |
| Amazon ECR | Store and deploy container images from Amazon Elastic Container Registry. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [amazon-fargate-openapi.yml](openapi/amazon-fargate-openapi.yml)

### JSON Schema

- [amazon-fargate-account-setting-schema.json](json-schema/amazon-fargate-account-setting-schema.json)
- [amazon-fargate-aws-vpc-configuration-schema.json](json-schema/amazon-fargate-aws-vpc-configuration-schema.json)
- [amazon-fargate-cluster-schema.json](json-schema/amazon-fargate-cluster-schema.json)
- [amazon-fargate-cluster-setting-schema.json](json-schema/amazon-fargate-cluster-setting-schema.json)
- [amazon-fargate-container-definition-schema.json](json-schema/amazon-fargate-container-definition-schema.json)
- [amazon-fargate-failure-schema.json](json-schema/amazon-fargate-failure-schema.json)
- [amazon-fargate-key-value-pair-schema.json](json-schema/amazon-fargate-key-value-pair-schema.json)
- [amazon-fargate-load-balancer-schema.json](json-schema/amazon-fargate-load-balancer-schema.json)
- [amazon-fargate-log-configuration-schema.json](json-schema/amazon-fargate-log-configuration-schema.json)
- [amazon-fargate-network-configuration-schema.json](json-schema/amazon-fargate-network-configuration-schema.json)
- [amazon-fargate-port-mapping-schema.json](json-schema/amazon-fargate-port-mapping-schema.json)
- [amazon-fargate-service-schema.json](json-schema/amazon-fargate-service-schema.json)
- [amazon-fargate-tag-schema.json](json-schema/amazon-fargate-tag-schema.json)
- [amazon-fargate-task-definition-schema.json](json-schema/amazon-fargate-task-definition-schema.json)
- [amazon-fargate-task-schema.json](json-schema/amazon-fargate-task-schema.json)

### JSON Structure

- [amazon-fargate-account-setting-structure.json](json-structure/amazon-fargate-account-setting-structure.json)
- [amazon-fargate-aws-vpc-configuration-structure.json](json-structure/amazon-fargate-aws-vpc-configuration-structure.json)
- [amazon-fargate-cluster-setting-structure.json](json-structure/amazon-fargate-cluster-setting-structure.json)
- [amazon-fargate-cluster-structure.json](json-structure/amazon-fargate-cluster-structure.json)
- [amazon-fargate-container-definition-structure.json](json-structure/amazon-fargate-container-definition-structure.json)
- [amazon-fargate-failure-structure.json](json-structure/amazon-fargate-failure-structure.json)
- [amazon-fargate-key-value-pair-structure.json](json-structure/amazon-fargate-key-value-pair-structure.json)
- [amazon-fargate-load-balancer-structure.json](json-structure/amazon-fargate-load-balancer-structure.json)
- [amazon-fargate-log-configuration-structure.json](json-structure/amazon-fargate-log-configuration-structure.json)
- [amazon-fargate-network-configuration-structure.json](json-structure/amazon-fargate-network-configuration-structure.json)
- [amazon-fargate-port-mapping-structure.json](json-structure/amazon-fargate-port-mapping-structure.json)
- [amazon-fargate-service-structure.json](json-structure/amazon-fargate-service-structure.json)
- [amazon-fargate-structure.json](json-structure/amazon-fargate-structure.json)
- [amazon-fargate-tag-structure.json](json-structure/amazon-fargate-tag-structure.json)
- [amazon-fargate-task-definition-structure.json](json-structure/amazon-fargate-task-definition-structure.json)
- [amazon-fargate-task-structure.json](json-structure/amazon-fargate-task-structure.json)

### Examples

- [amazon-fargate-account-setting-example.json](examples/amazon-fargate-account-setting-example.json)
- [amazon-fargate-aws-vpc-configuration-example.json](examples/amazon-fargate-aws-vpc-configuration-example.json)
- [amazon-fargate-cluster-example.json](examples/amazon-fargate-cluster-example.json)
- [amazon-fargate-cluster-setting-example.json](examples/amazon-fargate-cluster-setting-example.json)
- [amazon-fargate-container-definition-example.json](examples/amazon-fargate-container-definition-example.json)
- [amazon-fargate-example.json](examples/amazon-fargate-example.json)
- [amazon-fargate-failure-example.json](examples/amazon-fargate-failure-example.json)
- [amazon-fargate-key-value-pair-example.json](examples/amazon-fargate-key-value-pair-example.json)
- [amazon-fargate-load-balancer-example.json](examples/amazon-fargate-load-balancer-example.json)
- [amazon-fargate-log-configuration-example.json](examples/amazon-fargate-log-configuration-example.json)
- [amazon-fargate-network-configuration-example.json](examples/amazon-fargate-network-configuration-example.json)
- [amazon-fargate-port-mapping-example.json](examples/amazon-fargate-port-mapping-example.json)
- [amazon-fargate-service-example.json](examples/amazon-fargate-service-example.json)
- [amazon-fargate-tag-example.json](examples/amazon-fargate-tag-example.json)
- [amazon-fargate-task-definition-example.json](examples/amazon-fargate-task-definition-example.json)
- [amazon-fargate-task-example.json](examples/amazon-fargate-task-example.json)

### JSON-LD

- [amazon-fargate-context.jsonld](json-ld/amazon-fargate-context.jsonld)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [fargate.yaml](capabilities/shared/fargate.yaml)

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [amazon-fargate-container-orchestration.yaml](capabilities/amazon-fargate-container-orchestration.yaml) | Amazon Fargate API | 17 | Platform Engineer, DevOps Engineer |

## Vocabulary

- [Amazon Fargate Vocabulary](vocabulary/amazon-fargate-vocabulary.yaml) — Unified taxonomy mapping 7 resources, 10 actions, 1 workflow, and 3 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [amazon-fargate-spectral-rules.yml](rules/amazon-fargate-spectral-rules.yml) — 29 rules across 8 categories enforcing Amazon Fargate API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
