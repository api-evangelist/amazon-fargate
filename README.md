# Amazon Fargate

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Amazon Fargate is a serverless compute engine for containers that works with both Amazon ECS and Amazon EKS. Fargate removes the need to provision and manage servers, letting you specify and pay for resources per application, and improves security through application isolation by design.

**Areas:** AWS · Compute · Containers · ECS · EKS · Microservices · Serverless

## Amazon Fargate on APIs.io

Everything profiled for Amazon Fargate is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/amazon-fargate/](https://apis.io/providers/amazon-fargate/)** — or go straight to one of the APIs or properties below.

### APIs (6)

- [Amazon Fargate Account Settings API](https://apis.io/apis/amazon-fargate/amazon-fargate-account-settings-api/) — Operations for managing account-level settings
- [Amazon Fargate Clusters API](https://apis.io/apis/amazon-fargate/amazon-fargate-clusters-api/) — Operations for creating and managing ECS clusters
- [Amazon Fargate Services API](https://apis.io/apis/amazon-fargate/amazon-fargate-services-api/) — Operations for deploying and managing long-running services
- [Amazon Fargate Tagging API](https://apis.io/apis/amazon-fargate/amazon-fargate-tagging-api/) — Operations for tagging ECS resources
- [Amazon Fargate Task Definitions API](https://apis.io/apis/amazon-fargate/amazon-fargate-task-definitions-api/) — Operations for registering and managing task definitions
- [Amazon Fargate Tasks API](https://apis.io/apis/amazon-fargate/amazon-fargate-tasks-api/) — Operations for running and managing Fargate tasks

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### Arazzo workflows (9)

Multi-step workflows that chain operations into a business task.

- [Amazon Fargate Cluster Inventory Audit](https://apis.io/arazzos/amazon-fargate/amazon-fargate-cluster-inventory-audit-workflow/)
- [Amazon Fargate Decommission a Cluster](https://apis.io/arazzos/amazon-fargate/amazon-fargate-decommission-cluster-workflow/)
- [Amazon Fargate Deploy a Service](https://apis.io/arazzos/amazon-fargate/amazon-fargate-deploy-service-workflow/)
- [Amazon Fargate Provision and Run a Task](https://apis.io/arazzos/amazon-fargate/amazon-fargate-provision-and-run-task-workflow/)
- [Amazon Fargate Roll Out a Service Update](https://apis.io/arazzos/amazon-fargate/amazon-fargate-rolling-update-service-workflow/)
- [Amazon Fargate Run a Batch Task to Completion](https://apis.io/arazzos/amazon-fargate/amazon-fargate-run-batch-task-to-completion-workflow/)
- [Amazon Fargate Run the Latest Task Definition](https://apis.io/arazzos/amazon-fargate/amazon-fargate-run-latest-task-definition-workflow/)
- [Amazon Fargate Stop a Task and Confirm](https://apis.io/arazzos/amazon-fargate/amazon-fargate-stop-task-and-confirm-workflow/)
- [Amazon Fargate Tear Down a Service](https://apis.io/arazzos/amazon-fargate/amazon-fargate-teardown-service-workflow/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Amazon Fargate Agentic Access](https://apis.io/agentic-access/amazon-fargate/amazon-fargate-agentic-access/)

#### JSON Schema (15)

The data models the API reads and writes.

- [AccountSetting](https://apis.io/schemas/amazon-fargate/amazon-fargate-account-setting/)
- [AwsVpcConfiguration](https://apis.io/schemas/amazon-fargate/amazon-fargate-aws-vpc-configuration/)
- [Cluster](https://apis.io/schemas/amazon-fargate/amazon-fargate-cluster/)
- [ClusterSetting](https://apis.io/schemas/amazon-fargate/amazon-fargate-cluster-setting/)
- [ContainerDefinition](https://apis.io/schemas/amazon-fargate/amazon-fargate-container-definition/)
- [Failure](https://apis.io/schemas/amazon-fargate/amazon-fargate-failure/)
- [KeyValuePair](https://apis.io/schemas/amazon-fargate/amazon-fargate-key-value-pair/)
- [LoadBalancer](https://apis.io/schemas/amazon-fargate/amazon-fargate-load-balancer/)
- [LogConfiguration](https://apis.io/schemas/amazon-fargate/amazon-fargate-log-configuration/)
- [NetworkConfiguration](https://apis.io/schemas/amazon-fargate/amazon-fargate-network-configuration/)
- [PortMapping](https://apis.io/schemas/amazon-fargate/amazon-fargate-port-mapping/)
- [Service](https://apis.io/schemas/amazon-fargate/amazon-fargate-service/)
- [Tag](https://apis.io/schemas/amazon-fargate/amazon-fargate-tag/)
- [Task](https://apis.io/schemas/amazon-fargate/amazon-fargate-task/)
- [TaskDefinition](https://apis.io/schemas/amazon-fargate/amazon-fargate-task-definition/)

#### JSON Structure (16)

Data models expressed in JSON Structure.

- [Amazon Fargate Account Setting Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-account-setting-structure/)
- [Amazon Fargate Aws Vpc Configuration Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-aws-vpc-configuration-structure/)
- [Amazon Fargate Cluster Setting Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-cluster-setting-structure/)
- [Amazon Fargate Cluster Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-cluster-structure/)
- [Amazon Fargate Container Definition Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-container-definition-structure/)
- [Amazon Fargate Failure Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-failure-structure/)
- [Amazon Fargate Key Value Pair Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-key-value-pair-structure/)
- [Amazon Fargate Load Balancer Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-load-balancer-structure/)
- [Amazon Fargate Log Configuration Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-log-configuration-structure/)
- [Amazon Fargate Network Configuration Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-network-configuration-structure/)
- [Amazon Fargate Port Mapping Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-port-mapping-structure/)
- [Amazon Fargate Service Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-service-structure/)
- [Amazon Fargate Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-structure/)
- [Amazon Fargate Tag Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-tag-structure/)
- [Amazon Fargate Task Definition Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-task-definition-structure/)
- [Amazon Fargate Task Structure](https://apis.io/json-structure/amazon-fargate/amazon-fargate-task-structure/)

#### JSON-LD (1)

Linked-data contexts for the provider's vocabulary.

- [Amazon Fargate Context](https://apis.io/jsonld/amazon-fargate/amazon-fargate-context/)

#### Vocabulary (1)

The provider's domain terms, defined.

- [Amazon Fargate Vocabulary](https://apis.io/vocabularies/amazon-fargate/amazon-fargate-vocabulary/)

#### Examples (16)

Example requests and responses.

- [Amazon Fargate Account Setting Example](https://apis.io/examples/amazon-fargate/amazon-fargate-account-setting-example/)
- [Amazon Fargate Aws Vpc Configuration Example](https://apis.io/examples/amazon-fargate/amazon-fargate-aws-vpc-configuration-example/)
- [Amazon Fargate Cluster Example](https://apis.io/examples/amazon-fargate/amazon-fargate-cluster-example/)
- [Amazon Fargate Cluster Setting Example](https://apis.io/examples/amazon-fargate/amazon-fargate-cluster-setting-example/)
- [Amazon Fargate Container Definition Example](https://apis.io/examples/amazon-fargate/amazon-fargate-container-definition-example/)
- [Amazon Fargate Example](https://apis.io/examples/amazon-fargate/amazon-fargate-example/)
- [Amazon Fargate Failure Example](https://apis.io/examples/amazon-fargate/amazon-fargate-failure-example/)
- [Amazon Fargate Key Value Pair Example](https://apis.io/examples/amazon-fargate/amazon-fargate-key-value-pair-example/)
- [Amazon Fargate Load Balancer Example](https://apis.io/examples/amazon-fargate/amazon-fargate-load-balancer-example/)
- [Amazon Fargate Log Configuration Example](https://apis.io/examples/amazon-fargate/amazon-fargate-log-configuration-example/)
- [Amazon Fargate Network Configuration Example](https://apis.io/examples/amazon-fargate/amazon-fargate-network-configuration-example/)
- [Amazon Fargate Port Mapping Example](https://apis.io/examples/amazon-fargate/amazon-fargate-port-mapping-example/)
- [Amazon Fargate Service Example](https://apis.io/examples/amazon-fargate/amazon-fargate-service-example/)
- [Amazon Fargate Tag Example](https://apis.io/examples/amazon-fargate/amazon-fargate-tag-example/)
- [Amazon Fargate Task Definition Example](https://apis.io/examples/amazon-fargate/amazon-fargate-task-definition-example/)
- [Amazon Fargate Task Example](https://apis.io/examples/amazon-fargate/amazon-fargate-task-example/)

#### Collections (8)

Postman and Open Collection request collections.

- [Amazon Fargate API](https://apis.io/collections/amazon-fargate/open-amazon-fargate/)
- [Amazon Fargate Account Settings API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-account-settings-api/)
- [Amazon Fargate Account Settings Clusters API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-clusters-api/)
- [Amazon Fargate Account Settings Services API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-services-api/)
- [Amazon Fargate Account Settings Tagging API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-tagging-api/)
- [Amazon Fargate Account Settings Task Definitions API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-task-definitions-api/)
- [Amazon Fargate Account Settings Tasks API](https://apis.io/collections/amazon-fargate/open-amazon-fargate-tasks-api/)
- [Amazon Fargate API](https://apis.io/collections/amazon-fargate/postman-amazon-fargate/)

#### Governance rules (2)

Spectral rulesets for the provider's contracts.

- [Amazon Fargate — jsonschema rules](https://apis.io/rules/amazon-fargate/amazon-fargate-jsonschema-spectral-rules/)
- [Amazon Fargate API Rules](https://apis.io/rules/amazon-fargate/amazon-fargate-spectral-rules/)

#### Security (3)

Authentication, trust centre, and vulnerability disclosure.

- [Amazon Fargate Authentication](https://apis.io/security/amazon-fargate/amazon-fargate-authentication/)
- [Amazon Fargate Domain Security](https://apis.io/security/amazon-fargate/amazon-fargate-domain-security/)
- [Amazon Fargate Vulnerability Disclosure](https://apis.io/security/amazon-fargate/amazon-fargate-vulnerability-disclosure/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Amazon Fargate Plans Pricing](https://apis.io/plans/amazon-fargate/amazon-fargate-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Amazon Fargate Rate Limits](https://apis.io/rate-limits/amazon-fargate/amazon-fargate-rate-limits/)

#### FinOps (1)

What the API costs to run, by operation.

- [Amazon Fargate Finops](https://apis.io/finops/amazon-fargate/amazon-fargate-finops/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://aws.amazon.com/fargate/)
- [Documentation](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/containers/)
- [Terms of service](https://aws.amazon.com/service-terms/)
- [Privacy policy](https://aws.amazon.com/privacy/)
- [GitHub](https://github.com/aws)
- [Status](https://status.aws.amazon.com/)

## API Evangelist rating

**Kin Score 49.1** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 78.6 |
| Contract quality | 31.8 |
| Contract governance | 27.3 |
| Operational transparency | 36.8 |
| Developer ergonomics | 65.5 |
| Access clarity | 52.6 |

**Agent readiness 30.2** — agent-ready

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | — |
| MCP Server | — |
| Auth Clarity | bearer (partial credit) |
| Idempotency | — |
| Error Semantics | verified |
| OpenAPI Examples | verified |
| Rate Limit Signal | derived (partial credit) |
| Event Surface Described | — |
| Agent Skills | — |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | distinct |

**Regulatory alignment 30.4** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/amazon-fargate/](https://apis.io/providers/amazon-fargate/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Amazon Fargate: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/amazon-fargate/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
