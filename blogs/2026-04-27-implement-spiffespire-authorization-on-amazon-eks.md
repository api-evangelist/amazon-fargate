---
title: "Implement SPIFFE/SPIRE authorization on Amazon EKS"
url: "https://aws.amazon.com/blogs/containers/implement-spiffe-spire-authorization-on-amazon-eks/"
date: "Mon, 27 Apr 2026 17:29:44 +0000"
author: "Eric Cavalcanti"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>When running distributed applications across multiple <a href="https://github.com/spiffe/helm-charts-hardened" rel="noopener" target="_blank">Amazon Elastic Kubernetes Service (Amazon EKS)</a> clusters, teams face two critical security challenges: establishing secure communication in untrusted networks and authenticating workloads without relying on network-based controls.</p> 
<p><strong>The mTLS Challenge</strong>: Traditional approaches rely on network security to determine message sender identity and ensure message integrity. However, in complex distributed applications spanning multiple networks with services deployed by different teams, network-based protection becomes insufficient. Teams need mutual Transport Layer Security (mTLS) to cryptographically prove sender and recipient identity while ensuring messages haven’t been viewed or modified in transit.</p> 
<p><strong>The Authentication Challenge</strong>: While mTLS secures channels between specific workloads, teams also need flexible authentication for scenarios where direct mTLS isn’t possible—such as when Layer 7 load balancers sit between services, or when multiple workloads send messages over a single encrypted channel. JSON Web Tokens (JWTs) provide this flexibility but require proper key management and validation.</p> 
<p>SPIFFE (Secure Production Identity Framework for Everyone) and its reference implementation SPIRE (SPIFFE Runtime Environment) help address both challenges by:</p> 
<ul> 
 <li><strong>Attesting workload identity</strong> at runtime in distributed systems</li> 
 <li><strong>Delivering short-lived, automatically rotated X.509 certificates</strong> (X.509-SVIDs) for mTLS directly to workloads</li> 
 <li><strong>Generating and validating JWT-SVIDs</strong> for flexible authentication scenarios</li> 
 <li><strong>Minimizing manual certificate management</strong> through the SPIFFE Workload API</li> 
</ul> 
<p>This guide demonstrates deploying SPIRE in a nested configuration across multiple Amazon EKS clusters, enabling secure workload identity and authentication in distributed environments.</p> 
<p>In this post, we show you how to implement SPIFFE/SPIRE on Amazon EKS to establish secure service-to-service communication using a nested architecture. You’ll learn how to deploy SPIRE across multiple Amazon EKS clusters, configure workload attestation, and implement fine-grained authorization policies that scale with your infrastructure.</p> 
<h2>Understanding SPIFFE/SPIRE and Service-to-Service Authorization</h2> 
<p>Before diving into the implementation, let’s establish the foundational concepts of SPIFFE and SPIRE, and understand why they’re essential for modern service-to-service authorization.</p> 
<h3>What is SPIFFE?</h3> 
<p><a href="https://spiffe.io/docs/latest/spiffe-about/overview/" rel="noopener" target="_blank">SPIFFE</a> is a set of open-source standards for securely identifying software systems in dynamic, heterogeneous environments. It provides a framework for workloads to present cryptographically verifiable identities, enabling secure communication without relying on network-level controls like firewalls or IP whitelists.</p> 
<h3>What is SPIRE?</h3> 
<p><a href="https://spiffe.io/docs/latest/spire-about/" rel="noopener" target="_blank">SPIRE</a> is the reference implementation of the SPIFFE standards. It serves as a production-ready SPIFFE Runtime Environment that manages the lifecycle of SPIFFE identities, including attestation, registration, and key rotation. SPIRE can be deployed in various modes, including nested hierarchies for complex, multi-cluster environments.</p> 
<p><strong>Why implement SPIFFE/SPIRE?</strong> In dynamic containerized environments, traditional security approaches break down:</p> 
<ul> 
 <li><strong>IP-based security fails</strong> when containers move between nodes and clusters</li> 
 <li><strong>Shared secrets don’t scale</strong> across hundreds of microservices</li> 
 <li><strong>Manual certificate management</strong> becomes operationally complex</li> 
 <li><strong>Network perimeters blur</strong> in multi-cloud and hybrid deployments</li> 
</ul> 
<p>SPIRE can help address these challenges by providing <strong>cryptographic identity</strong> that moves with workloads, enabling <strong>automatic credential rotation</strong>, and supporting <strong>zero-trust authentication</strong> that is designed to work across different network locations.</p> 
<h3>The Need for Service-to-Service Authorization</h3> 
<p>Traditional security models often rely on network policies or API gateways, which can be cumbersome and error-prone in microservices architectures. SPIFFE/SPIRE helps address this by:</p> 
<ul> 
 <li>Providing workload identities that are cryptographically verifiable</li> 
 <li>Enabling mutual TLS (mTLS) authentication between services</li> 
 <li>Supporting fine-grained authorization policies based on identity</li> 
 <li>Automating key rotation and certificate management</li> 
</ul> 
<h3>What is Nested SPIRE?</h3> 
<p>Nested SPIRE is a deployment architecture that allows SPIRE Servers to be “chained” together, enabling all servers to issue identities within the same trust domain. This means workloads across different clusters receive identity documents that can be verified against the same root keys, creating a unified security boundary.</p> 
<p><strong>How the Chaining Works:</strong></p> 
<ul> 
 <li>A <strong>SPIRE Agent is co-located</strong> with every downstream SPIRE Server</li> 
 <li>The downstream server <strong>obtains credentials via the Workload API</strong> from its local agent</li> 
 <li>These credentials are used to <strong>authenticate directly with the upstream SPIRE Server</strong></li> 
 <li>The upstream server issues an <strong>intermediate Certificate Authority (CA)</strong> to the downstream server</li> 
 <li>All servers in the chain are configured to <strong>issue SVIDs within the same trust domain</strong></li> 
</ul> 
<p><strong>Key Benefits:</strong></p> 
<ul> 
 <li><strong>Unified Trust Domain</strong>: All workloads across clusters share the same cryptographic root of trust</li> 
 <li><strong>Multi-Cloud Ready</strong>: Different child servers can use different node attestors for various cloud environments</li> 
 <li><strong>Scalable Architecture</strong>: New clusters can be added by chaining additional SPIRE servers</li> 
 <li><strong>Centralized Root Management</strong>: Single root server manages the trust domain while child servers handle local workloads</li> 
</ul> 
<p>This architecture is particularly well-suited for multi-cloud deployments where workloads need to authenticate across different infrastructure environments while maintaining a consistent security model.</p> 
<h3>Architecture Overview</h3> 
<p><img alt="Architecture diagram showing a hierarchical SPIRE (Secure Production Identity Runtime Environment) deployment on AWS, with a root cluster managing two child clusters inside a private VPC subnet, backed by Amazon Aurora MySQL" class="alignnone size-full wp-image-20629" height="1031" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/18/CONTAINERS-75-spire-1-1.png" width="1430" /></p> 
<p>Figure 1: Nested SPIRE architecture showing root and child servers across multiple EKS clusters</p> 
<h3>Components</h3> 
<ul> 
 <li><strong>Root Cluster</strong> (spire-root-cluster): Hosts the root SPIRE server</li> 
 <li><strong>Child Clusters</strong> (spire-child-cluster-01, spire-child-cluster-02): Host child SPIRE servers and workloads</li> 
 <li><strong><a href="https://aws.amazon.com/rds/aurora/" rel="noopener" target="_blank">Amazon Aurora</a></strong>: Persistent datastore for the root SPIRE server</li> 
 <li><strong>Kubeconfig Generator</strong>: Script to create secure cluster access configurations</li> 
</ul> 
<h2>Node Attestation in Nested SPIRE</h2> 
<p>Before diving into the deployment process, it’s crucial to understand how SPIRE establishes trust between servers and agents across clusters through <strong>node attestation</strong>.</p> 
<h3>What is Node Attestation?</h3> 
<p>Node attestation is the process where each SPIRE agent authenticates and verifies itself when first connecting to a SPIRE server. During this process, the agent and server work together to verify the identity of the node the agent is running on using plugins called <strong>node attestors</strong>.</p> 
<p>Node attestors interrogate a node and its environment for information that only that specific node would possess, proving the node’s identity through various methods:</p> 
<ul> 
 <li><strong>Cloud platform identity documents</strong> (such as AWS Instance Identity Documents)</li> 
 <li><strong>Hardware Security Module or TPM private keys</strong></li> 
 <li><strong>Manual verification via join tokens</strong></li> 
 <li><strong>Multi-node software system credentials</strong> (such as Kubernetes Service Account tokens)</li> 
 <li><strong>Deployed server certificates</strong></li> 
</ul> 
<p>Upon successful node attestation, the SPIRE server issues a unique <strong>SPIFFE ID</strong> to the agent, which serves as the “parent” identity for all workloads it manages.</p> 
<h3>Kubernetes Node Attestation (k8s_psat)</h3> 
<p>In our Nested SPIRE deployment, we use the <strong><a href="https://github.com/spiffe/spire/blob/main/doc/plugin_server_nodeattestor_k8s_psat.md" rel="noopener" target="_blank">k8s_psat plugin</a></strong> for node attestation. This plugin:</p> 
<ol> 
 <li><strong>Validates signed projected service account tokens</strong> provided by agents</li> 
 <li><strong>Uses the <a href="https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-review-v1/" rel="noopener" target="_blank">Kubernetes Token Review API</a></strong> to perform validation</li> 
 <li><strong>Queries the Kubernetes API server</strong> for additional node metadata (UID, namespace, service account)</li> 
 <li><strong>Generates SPIFFE IDs</strong> based on the validated node information</li> 
</ol> 
<h3>Why Cross-Cluster Access is Required</h3> 
<p>In a nested architecture, the <strong>root SPIRE server must validate tokens from child cluster nodes</strong>. This is why our deployment process includes:</p> 
<ol> 
 <li><strong>Kubeconfig generation script</strong> – Creates secure access credentials for each child cluster</li> 
 <li><strong>Base64-encoded kubeconfig injection</strong> – It provides the root server with child cluster API access: <pre><code class="language-bash">--set "external-spire-server.kubeConfigs.child01.kubeConfigBase64=$(cat ../script/spire-child-cluster-01.kubeconfig)"
</code></pre> </li> 
</ol> 
<p>This cross-cluster access enables the root server to perform Kubernetes Token Review API calls against child clusters, validating that agents requesting intermediate CA certificates are legitimate nodes within the trusted infrastructure.</p> 
<h2>Prerequisites and Environment Setup</h2> 
<p>Before deploying SPIFFE/SPIRE on Amazon EKS, verify you have the following prerequisites:</p> 
<h3>Required Tools and Software</h3> 
<ul> 
 <li><strong>AWS CLI</strong>: Version 2.32.0 or newer (<a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">installation guide</a>), configured with appropriate credentials and permissions</li> 
 <li><strong>Terraform</strong>: Version 1.12.2 or newer (<a href="https://developer.hashicorp.com/terraform/install" rel="noopener" target="_blank">installation guide</a>) for infrastructure provisioning</li> 
 <li><strong>kubectl</strong>: Version 1.34 or newer (<a href="https://kubernetes.io/docs/tasks/tools/#kubectl" rel="noopener" target="_blank">installation guide</a>) for Kubernetes cluster management</li> 
 <li><strong>Helm</strong>: Version 3.12.2 (v3 only, not v4) (<a href="https://helm.sh/docs/v3/intro/install" rel="noopener" target="_blank">installation guide</a>) for package management</li> 
 <li><strong>kubectx</strong>: Version 0.9.5 (<a href="https://github.com/ahmetb/kubectx/?tab=readme-ov-file#installation" rel="noopener" target="_blank">installation guide</a>) for cluster context switching</li> 
</ul> 
<h3>AWS Account Requirements</h3> 
<p>You’ll need an AWS account with permissions to:</p> 
<ul> 
 <li>Create EKS clusters</li> 
 <li>Manage VPCs and subnets</li> 
 <li>Deploy IAM roles and policies</li> 
</ul> 
<h3>Infrastructure Components</h3> 
<p>Our Nested SPIRE deployment consists of several key AWS infrastructure components that work together to provide secure workload identity across multiple clusters.</p> 
<h4>1. EKS Clusters</h4> 
<p>The deployment consists of three EKS clusters:</p> 
<table border="1px" cellpadding="10px"> 
 <thead> 
  <tr> 
   <th>Cluster</th> 
   <th>Purpose</th> 
   <th>Terraform Path</th> 
   <th>Region</th> 
   <th>Node Groups</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td><strong>spire-root-cluster</strong></td> 
   <td>Hosts the root SPIRE server</td> 
   <td><code>infrastructure/eks/spire-root-cluster/</code></td> 
   <td>us-east-1</td> 
   <td>Optimized for SPIRE server workloads</td> 
  </tr> 
  <tr> 
   <td><strong>spire-child-cluster-01</strong></td> 
   <td>Hosts child SPIRE server and application workloads</td> 
   <td><code>infrastructure/eks/spire-child-cluster-01/</code></td> 
   <td>us-east-1</td> 
   <td>Mixed instance types for diverse workloads</td> 
  </tr> 
  <tr> 
   <td><strong>spire-child-cluster-02</strong></td> 
   <td>Hosts child SPIRE server and application workloads</td> 
   <td><code>infrastructure/eks/spire-child-cluster-02/</code></td> 
   <td>us-east-1</td> 
   <td>Mixed instance types for diverse workloads</td> 
  </tr> 
 </tbody> 
</table> 
<h4>2. Amazon Aurora MySQL Datastore (<code>infrastructure/rds/spire-datastore/</code>)</h4> 
<p>The root SPIRE server requires persistent storage for audit logs, registration entries, node attestation data, and certificate authority information.</p> 
<p><strong>Configuration:</strong></p> 
<ul> 
 <li><strong>Engine</strong>: MySQL 8.0</li> 
 <li><strong>Instance Class</strong>: Serverless v2 (auto-scaling)</li> 
 <li><strong>Storage</strong>: Encrypted with AWS KMS</li> 
 <li><strong>Backup</strong>: Automated daily backups</li> 
 <li><strong>Multi-AZ</strong>: Enabled for high availability</li> 
</ul> 
<h4>3. Networking Architecture</h4> 
<p>Each cluster is deployed in its own or shared VPC with:</p> 
<ul> 
 <li><strong>Public Subnets</strong>: For load balancers and NAT gateways</li> 
 <li><strong>Private Subnets</strong>: For EKS nodes and Amazon Aurora instances</li> 
 <li><strong>Security Groups</strong>: Restrictive rules for inter-cluster communication</li> 
 <li><strong>VPC Peering</strong>: Enables secure communication between clusters</li> 
</ul> 
<h3>Environment Variables</h3> 
<p>Set the following environment variables:</p> 
<pre><code class="language-bash">export AWS_REGION=your-aws-region # e.g., us-east-1
export CLUSTER_NAME_ROOT=your-root-cluster-name # e.g., spire-root-cluster
export CLUSTER_NAME_CHILD1=your-child-cluster-1-name # e.g., spire-child-cluster-01
export CLUSTER_NAME_CHILD2=your-child-cluster-2-name # e.g., spire-child-cluster-02
</code></pre> 
<h2>Deploying Amazon EKS Infrastructure with Terraform</h2> 
<p>Now that we understand the architecture and components, let’s deploy the AWS infrastructure using Terraform to create our Nested SPIRE environment.</p> 
<h3>Infrastructure Overview</h3> 
<p>We’ll deploy three EKS clusters: a root cluster for the SPIRE trust anchor and two child clusters for workload deployment.</p> 
<h3>Deployment Steps</h3> 
<p>Follow these steps to deploy the infrastructure components, starting with networking, then the database, and finally the EKS clusters.</p> 
<h4>1. Obtain Networking Values</h4> 
<p>Before deploying the SPIRE infrastructure, you’ll need to gather the following networking values from your existing VPC setup:</p> 
<ul> 
 <li>VPC ID</li> 
 <li>Private subnets (list)</li> 
 <li>Public subnets (list)</li> 
 <li>VPC CIDR block</li> 
 <li>Route 53 DNS zone (for external-dns to create SPIRE server domain pointing to ALB)</li> 
</ul> 
<p>These values will be required for the database and EKS cluster <code>variables.tf</code> files in the subsequent deployment steps.</p> 
<h4>2. Deploy Aurora MySQL</h4> 
<p>Update the <code>variables.tf</code> file with the networking values from the previous step.</p> 
<pre><code class="language-bash">cd infrastructure/rds/spire-datastore/
terraform init
terraform plan
terraform apply
</code></pre> 
<p>After deployment, retrieve the database endpoint and secrets ARN that will be used in subsequent configurations:</p> 
<pre><code class="language-bash">terraform output aurora_mysql_v2_cluster_endpoint
terraform output aurora_mysql_v2_cluster_master_user_secret_arn
</code></pre> 
<p>Save these values as they will be required for the root EKS cluster and <code>root-values.yaml</code> configuration in the SPIRE installation steps.</p> 
<h4>3. Deploy Root EKS Cluster</h4> 
<p>Update the <code>variables.tf</code> file with the networking values from step 1, the database values from step 2, your Route 53 DNS zone, and RDS cluster ARN.</p> 
<pre><code class="language-bash">cd ../../eks/spire-root-cluster/
terraform init
terraform plan
terraform apply
</code></pre> 
<p>After deployment, retrieve the IAM role ARNs that will be needed for the child cluster configurations:</p> 
<pre><code class="language-bash">terraform output cluster_iam_role_arn
terraform output node_iam_role_arn
terraform output spire_server_service_account_role_arn
</code></pre> 
<p>Save these ARN values as they will be required for the child cluster <code>variables.tf</code> files.</p> 
<h4>4. Deploy Child Cluster 01</h4> 
<p>Update the <code>variables.tf</code> file with the networking values from step 1, the database values from step 2, and the IAM role ARNs from step 3.</p> 
<pre><code class="language-bash">cd ../spire-child-cluster-01/
terraform init
terraform plan
terraform apply
</code></pre> 
<h4>5. Deploy Child Cluster 02</h4> 
<p>Update the <code>variables.tf</code> file with the networking values from step 1, the database values from step 2, and the IAM role ARNs from step 3.</p> 
<pre><code class="language-bash">cd ../spire-child-cluster-02/
terraform init
terraform plan
terraform apply
</code></pre> 
<h4>6. Setup Kubeconfigs Locally</h4> 
<p>Update your kubeconfig files for all three clusters:</p> 
<pre><code class="language-bash">aws eks --region us-east-1 update-kubeconfig --name spire-root-cluster
aws eks --region us-east-1 update-kubeconfig --name spire-child-cluster-01
aws eks --region us-east-1 update-kubeconfig --name spire-child-cluster-02
</code></pre> 
<h3>7. Run the Generate Kubeconfig Script</h3> 
<p>In the <code>script</code> directory, execute the script to generate base64-encoded kubeconfigs:</p> 
<pre><code class="language-bash">cd script
./generate_kubeconfig.sh
</code></pre> 
<p><strong>What this script does:</strong></p> 
<ol> 
 <li>Creates temporary kubeconfig files for each child cluster</li> 
 <li>Extracts cluster certificate authority data and endpoints</li> 
 <li>Retrieves service account tokens from the <code>spire-system</code> namespace</li> 
 <li>Generates base64-encoded kubeconfig files for secure cluster access</li> 
 <li>Creates both encoded and decoded versions for verification</li> 
</ol> 
<h2>Installing SPIRE Server on EKS</h2> 
<p>With the infrastructure deployed and kubeconfigs generated, we can now install SPIRE across our clusters using Helm charts.</p> 
<h3>Helm Values Structure</h3> 
<p>The SPIRE deployment uses multiple Helm values files to organize configuration settings for different cluster roles and shared parameters.</p> 
<h4><code>your-values.yaml</code></h4> 
<p>Contains common configuration shared across all clusters:</p> 
<ul> 
 <li>Image registry settings (for private registries)</li> 
 <li>Resource limits and requests</li> 
 <li>Security contexts</li> 
 <li>Logging configuration</li> 
</ul> 
<h4><code>root-values.yaml</code></h4> 
<p>Specific configuration for the root SPIRE server:</p> 
<ul> 
 <li>Database connection settings</li> 
 <li>Trust domain configuration</li> 
 <li>External cluster access settings</li> 
 <li>Certificate authority configuration</li> 
</ul> 
<h4><code>spire-child-cluster-01.yaml</code> and <code>spire-child-cluster-02.yaml</code></h4> 
<p>Child-specific configurations:</p> 
<ul> 
 <li>Parent server connection details</li> 
 <li>Local cluster settings</li> 
 <li>Node attestation configuration</li> 
</ul> 
<h3>Key Configuration Parameters</h3> 
<p><strong>Root Server Configuration:</strong></p> 
<pre><code class="language-yaml">spire-server:
  dataStore:
    sql:
      databaseType: mysql
      databaseName: sharedqaspireserver
      host: "demo-serverless-mysqlv2.cluster-abc123def456.us-east-1.rds.amazonaws.com"
      port: 3306
      region: "us-east-1"
      username: spireadmin
      externalSecret:
        enabled: true
        name: spire-db-secret
        key: password

  trustDomain: "spireserver.example.com"
  
  nodeAttestor:
    k8sPSAT:
      serviceAccountAllowList:
        - "spire-mgmt:spire-agent"
        - "default:default"
</code></pre> 
<p><strong>Child Server Configuration:</strong></p> 
<pre><code class="language-yaml">spire-server:
  upstreamAuthority:
    spire:
      enabled: true
      upstreamDriver: "spire-plugin"
      serverAddress: "spire-server.spire-mgmt.svc.cluster.local"
      serverPort: 8081
      
  trustDomain: "spireserver.example.com"
</code></pre> 
<h3>Critical Configuration Note: Service Name Length Constraints</h3> 
<p><strong>IMPORTANT</strong>: When deploying Nested SPIRE, the cluster identifiers used in Helm commands must be <strong>7 characters or fewer</strong>. This is due to Internet Assigned Numbers Authority (IANA) that constraints the limit of the service names to 15 characters maximum.</p> 
<p><a href="https://www.rfc-editor.org/rfc/rfc6335.html#section-5.1" rel="noopener" target="_blank">RFC 6335 Section 5.1</a>:</p> 
<ul> 
 <li>Valid service names are hereby normatively defined as follows: 
  <ul> 
   <li>MUST be at least 1 character and no more than 15 characters long</li> 
  </ul> </li> 
</ul> 
<p>The SPIRE Helm chart appends these identifiers to service names, causing deployment failures if too long.</p> 
<ul> 
 <li>Template generates: <code>prom-cm-{CLUSTER_ID}</code></li> 
 <li>With <code>ABCDEFG-qa-01</code>: becomes <code>prom-cm-ABCDEFG-qa-01</code> (21 characters) <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/14.0.0/72x72/274c.png" style="height: 1em;" /></li> 
 <li>With <code>child01</code>: becomes <code>prom-cm-child01</code> (15 characters) <img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/14.0.0/72x72/2705.png" style="height: 1em;" /></li> 
</ul> 
<p><strong>Example of the constraint:</strong></p> 
<pre><code class="language-bash"># ❌ This will FAIL - "ABCDEFG-qa-01" is too long (12 characters)
--set "external-spire-server.kubeConfigs.ABCDEFG-qa-01.kubeConfigBase64=..."

# ✅ This will WORK - "child01" is short enough (7 characters)  
--set "external-spire-server.kubeConfigs.child01.kubeConfigBase64=..."
</code></pre> 
<p>The cluster identifier (e.g., <code>child01</code>) is:</p> 
<ul> 
 <li>Used only for Helm configuration – it doesn’t need to match the actual EKS cluster name</li> 
 <li>Appended to service names by the Helm template</li> 
 <li>Must comply with RFC 6335 naming constraints</li> 
</ul> 
<h3>1: Setup Root Cluster spire-root-cluster</h3> 
<p>We’ll start by configuring the root cluster, which serves as the trust anchor for our Nested SPIRE deployment.</p> 
<h4>1a. Use the Root Cluster Context</h4> 
<p>Switch to the root cluster context:</p> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-root-cluster
</code></pre> 
<h4>1b. Install CRDs on the Root Cluster</h4> 
<p>Navigate back to the <code>helm</code> directory and install the CRDs on the root cluster:</p> 
<pre><code class="language-bash">cd ../helm
helm upgrade --install --create-namespace -n spire-mgmt spire-crds spire-crds \
--repo https://spiffe.github.io/helm-charts-hardened/
</code></pre> 
<h4>1c. Install the Root Server</h4> 
<p>Install the root server using the encoded kubeconfigs for child clusters:</p> 
<pre><code class="language-bash">helm upgrade --install -n spire-mgmt spire spire-nested --repo https://spiffe.github.io/helm-charts-hardened/ \
--set "external-spire-server.kubeConfigs.child01.kubeConfigBase64=$(cat ../script/spire-child-cluster-01.kubeconfig)" \
--set "external-spire-server.kubeConfigs.child02.kubeConfigBase64=$(cat ../script/spire-child-cluster-02.kubeconfig)" \
-f your-values.yaml -f root-values.yaml
</code></pre> 
<p><strong>Command Explanation:</strong></p> 
<ul> 
 <li><code>child01</code> and <code>child02</code> are short identifiers (≤7 chars) to avoid service name length issues</li> 
 <li>These identifiers are used internally by Helm and don’t need to match cluster names</li> 
 <li>The actual cluster access is provided by the base64-encoded kubeconfig files</li> 
 <li>Each <code>kubeConfigBase64</code> parameter contains the complete authentication information for accessing the respective child cluster</li> 
</ul> 
<h3>2. Setup Child Cluster spire-child-cluster-01</h3> 
<p>Now we’ll configure the first child cluster, which will connect to the root server to obtain its intermediate CA certificate.</p> 
<h4>2a. Switch to spire-child-cluster-01 Cluster</h4> 
<p>Switch to the <code>spire-child-cluster-01</code> cluster context:</p> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-child-cluster-01
</code></pre> 
<h4>2b. Mark spire-system namespace as Helm-managed for the spire release</h4> 
<pre><code class="language-bash">kubectl label namespace spire-system app.kubernetes.io/managed-by=Helm --overwrite &amp;&amp; kubectl annotate namespace spire-system meta.helm.sh/release-name=spire meta.helm.sh/release-namespace=spire-mgmt --overwrite
</code></pre> 
<h4>2c. Install CRDs and Server for spire-child-cluster-01</h4> 
<p>Install the CRDs and server for <code>spire-child-cluster-01</code>:</p> 
<pre><code class="language-bash">helm upgrade --install --create-namespace -n spire-mgmt spire-crds spire-crds \
--repo https://spiffe.github.io/helm-charts-hardened/
helm upgrade --install -n spire-mgmt spire spire-nested --repo https://spiffe.github.io/helm-charts-hardened/ \
-f your-values.yaml -f spire-child-cluster-01.yaml
</code></pre> 
<h3>3. Setup Child Cluster spire-child-cluster-02</h3> 
<p>Next, we’ll configure the second child cluster using the same process as the first child cluster.</p> 
<h4>3a. Switch to spire-child-cluster-02 Cluster</h4> 
<p>Switch to the <code>spire-child-cluster-02</code> cluster context:</p> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-child-cluster-02
</code></pre> 
<h4>3b. Mark spire-system namespace as Helm-managed for the spire release</h4> 
<pre><code class="language-bash">kubectl label namespace spire-system app.kubernetes.io/managed-by=Helm --overwrite &amp;&amp; kubectl annotate namespace spire-system meta.helm.sh/release-name=spire meta.helm.sh/release-namespace=spire-mgmt --overwrite
</code></pre> 
<h4>3c. Install CRDs and Server for spire-child-cluster-02</h4> 
<p>Install the CRDs and server for <code>spire-child-cluster-02</code> using the specific child values file:</p> 
<pre><code class="language-bash">helm upgrade --install --create-namespace -n spire-mgmt spire-crds spire-crds \
--repo https://spiffe.github.io/helm-charts-hardened/
helm upgrade --install -n spire-mgmt spire spire-nested --repo https://spiffe.github.io/helm-charts-hardened/ \
-f your-values.yaml -f spire-child-cluster-02.yaml
</code></pre> 
<h2>Deploy Envoy Test Application</h2> 
<p>With SPIRE installed across all clusters, we’ll now deploy a test application to demonstrate secure service-to-service communication using SPIFFE identities.</p> 
<h3>1. Switch to spire-child-cluster-01</h3> 
<p>Ensure you’re connected to the child cluster:</p> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-child-cluster-01
</code></pre> 
<h3>2. Update SPIFFE Trust Domain in ConfigMaps</h3> 
<p>Before deploying, update the trust domain in each <code>configmap.yaml</code> file under the <code>envoy/</code> directory to match your root cluster’s trust domain.</p> 
<p>For example, change:</p> 
<pre><code class="language-yaml">- name: "spiffe://spirenested.example.com/ns/ecommerce/sa/edge-proxy-service-account"
</code></pre> 
<p>The domain must match the <code>trustDomain</code> value in <code>helm/root-values.yaml</code>:</p> 
<pre><code class="language-yaml">trustDomain: spirenested.example.com
</code></pre> 
<h3>3. Create Namespace and Deploy</h3> 
<p>Create the ecommerce namespace and deploy the application:</p> 
<pre><code class="language-bash">kubectl create ns ecommerce
kubectl apply -R -f envoy/
</code></pre> 
<h3>4. Get Network Load Balancer DNS</h3> 
<p>Wait for the NLB to provision, then retrieve its DNS:</p> 
<pre><code class="language-bash">kubectl get svc -n ecommerce edge-proxy -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
</code></pre> 
<h3>5. Test the Application</h3> 
<p>Access the GraphQL endpoint using the NLB DNS:</p> 
<pre><code class="language-text">http://&lt;NLB-DNS&gt;:8081/v1/graphql
</code></pre> 
<p>Example:</p> 
<pre><code class="language-text">http://k8s-ecommerc-edgeprox-a1b2c3d4e5-f6g7h8i9j0k1l2m3.elb.us-east-1.amazonaws.com:8081/v1/graphql
</code></pre> 
<p>Run this query to verify the deployment:</p> 
<pre><code class="language-graphql">query {
  orders {
    id
    orderFor
    product {
      id
      name
    }
  },
  products {
    id
    name
  }
}
</code></pre> 
<p>You should see data for orders and products returned successfully.</p> 
<h3>6. Introduce a SPIFFE ID Mismatch</h3> 
<p>Edit <code>envoy/graphql/configmap.yaml</code> line 182 and change the SPIFFE ID from plural to singular:</p> 
<pre><code class="language-yaml">exact: "spiffe://spirenested.example.com/ns/ecommerce/sa/order-service-account"
</code></pre> 
<p>Apply the change and restart the deployment:</p> 
<pre><code class="language-bash">kubectl apply -f envoy/graphql/configmap.yaml
kubectl rollout restart deployment graphql -n ecommerce
</code></pre> 
<h3>7. Observe the Error</h3> 
<p>Run the same GraphQL query. You’ll see an error for orders:</p> 
<pre><code class="language-json">{
  "errors": [
    {
      "message": "Request failed with status code 503",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "orders"
      ]
    }
  ]
}
</code></pre> 
<p>This occurs because the SPIFFE ID no longer matches the orders service.</p> 
<h3>8. Fix the Issue</h3> 
<p>Revert the change back to the correct SPIFFE ID:</p> 
<pre><code class="language-yaml">exact: "spiffe://spirenested.example.com/ns/ecommerce/sa/orders-service-account"
</code></pre> 
<p>Apply and restart:</p> 
<pre><code class="language-bash">kubectl apply -f envoy/graphql/configmap.yaml
kubectl rollout restart deployment graphql -n ecommerce
</code></pre> 
<p>The query should now work correctly again.</p> 
<h3>Common Issues and Solutions</h3> 
<h4>1. Service Name Length Constraints</h4> 
<p><strong>Problem</strong>: Helm template generates service names exceeding 15 characters (RFC 6335 limit) <strong>Error</strong>: <code>must be no more than 15 characters</code></p> 
<p><strong>Root Cause</strong>: The SPIRE Helm chart appends cluster identifiers to service names. For example:</p> 
<ul> 
 <li>Template generates: <code>prom-cm-{CLUSTER_ID}</code></li> 
 <li>With <code>ABCDEFG-qa-01</code>: becomes <code>prom-cm-ABCDEFG-qa-01</code> (21 characters) <img alt="❌" class="wp-smiley" src="https://s.w.org/images/core/emoji/14.0.0/72x72/274c.png" style="height: 1em;" /></li> 
 <li>With <code>child01</code>: becomes <code>prom-cm-child01</code> (15 characters) <img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/14.0.0/72x72/2705.png" style="height: 1em;" /></li> 
</ul> 
<p><strong>Solution</strong>: Use cluster identifiers of 7 characters or fewer:</p> 
<pre><code class="language-bash"># ❌ Too long - will cause deployment failure
--set "external-spire-server.kubeConfigs.routing-qa-01.kubeConfigBase64=..."

# ✅ Correct - short identifier
--set "external-spire-server.kubeConfigs.child01.kubeConfigBase64=..."
</code></pre> 
<p><strong>Important</strong>: The cluster identifier is only used for Helm templating and doesn’t need to match the actual EKS cluster name.</p> 
<h4>2. Image Registry Issues</h4> 
<p><strong>Problem</strong>: Cannot pull images from public registries <strong>Solution</strong>: Configure private registry in <code>your-values.yaml</code>:</p> 
<pre><code class="language-yaml">global:
  spire:
    image:
      registry: "your-private-registry.com"
</code></pre> 
<h4>3. Storage Class Issues</h4> 
<p><strong>Problem</strong>: <code>pod has unbound immediate PersistentVolumeClaims</code> <strong>Solution</strong>: Ensure default storage class is configured:</p> 
<pre><code class="language-bash">kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
</code></pre> 
<h4>4. IP Exhaustion</h4> 
<p><strong>Problem</strong>: Not enough IP addresses in VPC CIDR <strong>Solution</strong>:</p> 
<ul> 
 <li>Use larger CIDR blocks for VPCs</li> 
 <li>Implement IP address management (IPAM)</li> 
 <li>Consider using AWS VPC CNI with prefix delegation</li> 
</ul> 
<h4>5. IMDSv2 Compatibility Issue</h4> 
<p>SPIRE does not currently support IMDSv2 when strictly enforced. The Terraform configuration sets <code>http_tokens = "optional"</code> to allow both IMDSv1 and IMDSv2:</p> 
<pre><code class="language-hcl">eks_managed_node_groups = {
  general-instances = {
    ...
    metadata_options = {
      http_endpoint = "enabled"
      http_tokens   = "optional"
    }
  }
}
</code></pre> 
<p><strong>Security Implications</strong>: Allowing IMDSv1 increases the risk of Server-Side Request Forgery (SSRF) attacks, as IMDSv1 does not require session tokens. To mitigate this risk:</p> 
<ul> 
 <li>Implement network policies to restrict pod access to the metadata service</li> 
 <li>Monitor IMDSv1 usage through CloudWatch metrics and VPC Flow Logs</li> 
 <li>Apply defense-in-depth controls such as IAM role restrictions and security groups</li> 
 <li>Review the <a href="https://docs.aws.amazon.com/eks/latest/userguide/best-practices-security.html" rel="noopener" target="_blank">Amazon EKS security best practices guide</a> for additional recommendations</li> 
</ul> 
<p>For an extensive review of IMDSv2 security enhancements, see <a href="https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/" rel="noopener" target="_blank">Add defense in depth against open firewalls, reverse proxies, and SSRF vulnerabilities with enhancements to the EC2 Instance Metadata Service</a>. For more details on the SPIRE limitation, see: <a href="https://github.com/spiffe/spire/issues/6118" rel="noopener" target="_blank">IMDSv2 Requirement Breaks aws_mysql and aws_postgres Authentication</a></p> 
<h3>Verification Commands</h3> 
<p>Check SPIRE server status:</p> 
<pre><code class="language-bash">kubectl exec -n spire-mgmt spire-server-0 -- /opt/spire/bin/spire-server healthcheck
</code></pre> 
<p>List registration entries:</p> 
<pre><code class="language-bash">kubectl exec -n spire-mgmt spire-server-0 -- /opt/spire/bin/spire-server entry show
</code></pre> 
<p>Check agent status:</p> 
<pre><code class="language-bash">kubectl exec -n spire-mgmt daemonset/spire-agent -- /opt/spire/bin/spire-agent healthcheck
</code></pre> 
<h3>Troubleshooting Commands</h3> 
<p>Check SPIRE server logs:</p> 
<pre><code class="language-bash">kubectl logs -n spire-system deployment/spire-server
</code></pre> 
<p>Verify agent connectivity:</p> 
<pre><code class="language-bash">kubectl exec -it spire-agent-pod -- spire-agent api fetch
</code></pre> 
<p>Test mTLS communication:</p> 
<pre><code class="language-bash">openssl s_client -connect my-service:8443 -cert /path/to/cert.pem -key /path/to/key.pem
</code></pre> 
<h2>Best Practices and Security Considerations</h2> 
<h3>1. Infrastructure Management</h3> 
<ul> 
 <li>Use Infrastructure as Code (Terraform) for reproducible deployments</li> 
 <li>Implement proper tagging strategies for resource management</li> 
 <li>Use separate AWS accounts for different environments</li> 
</ul> 
<h3>2. Security</h3> 
<ul> 
 <li>Rotate certificates regularly</li> 
 <li>Implement comprehensive monitoring and alerting</li> 
 <li>Use AWS IAM roles for service accounts (IRSA)</li> 
 <li>Enable AWS CloudTrail for audit logging</li> 
</ul> 
<h3>3. Operations</h3> 
<ul> 
 <li>Implement automated backup strategies for Amazon Aurora</li> 
 <li>Use GitOps for configuration management</li> 
 <li>Implement proper CI/CD pipelines for updates</li> 
 <li>Monitor resource utilization and costs</li> 
</ul> 
<h3>4. Scalability</h3> 
<ul> 
 <li>Plan for cluster growth and additional child clusters</li> 
 <li>Implement horizontal pod autoscaling</li> 
 <li>Use cluster autoscaling for dynamic node management</li> 
 <li>Consider multi-region deployments for disaster recovery</li> 
</ul> 
<h3>5. Additional Security Considerations</h3> 
<ul> 
 <li><strong>Network Policies</strong>: Implement Kubernetes network policies to restrict traffic</li> 
 <li><strong>RBAC</strong>: Use least-privilege service accounts</li> 
 <li><strong>Secrets Management</strong>: Store sensitive data in AWS Secrets Manager</li> 
 <li><strong>Encryption</strong>: Enable encryption at rest and in transit</li> 
 <li><strong>Audit Logging</strong>: Enable comprehensive audit logging</li> 
</ul> 
<h2>Cleanup</h2> 
<p>Before destroying the Terraform infrastructure, you must first uninstall Helm releases to remove AWS resources created by Kubernetes controllers (such as Application Load Balancers, security groups, and IAM roles).</p> 
<h3>1. Uninstall Helm Releases from Child Cluster 02</h3> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-child-cluster-02
helm -n spire-mgmt uninstall spire
helm -n spire-mgmt uninstall spire-crds
kubectl -n spire-system delete pvc -l app.kubernetes.io/instance=spire
kubectl delete crds clusterfederatedtrustdomains.spire.spiffe.io clusterspiffeids.spire.spiffe.io clusterstaticentries.spire.spiffe.io
</code></pre> 
<h3>2. Uninstall Helm Releases from Child Cluster 01</h3> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-child-cluster-01
helm -n spire-mgmt uninstall spire
helm -n spire-mgmt uninstall spire-crds
kubectl -n spire-system delete pvc -l app.kubernetes.io/instance=spire
kubectl delete crds clusterfederatedtrustdomains.spire.spiffe.io clusterspiffeids.spire.spiffe.io clusterstaticentries.spire.spiffe.io
</code></pre> 
<h3>3. Uninstall Helm Releases from Root Cluster</h3> 
<pre><code class="language-bash">kubectx arn:aws:eks:us-east-1:111122223333:cluster/spire-root-cluster
helm -n spire-mgmt uninstall spire
helm -n spire-mgmt uninstall spire-crds
kubectl -n spire-mgmt delete pvc -l app.kubernetes.io/instance=spire
kubectl delete crds clusterfederatedtrustdomains.spire.spiffe.io clusterspiffeids.spire.spiffe.io clusterstaticentries.spire.spiffe.io
</code></pre> 
<h3>4. Destroy Child Cluster 02</h3> 
<pre><code class="language-bash">cd infrastructure/eks/spire-child-cluster-02/
terraform destroy
</code></pre> 
<h3>5. Destroy Child Cluster 01</h3> 
<pre><code class="language-bash">cd ../spire-child-cluster-01/
terraform destroy
</code></pre> 
<h3>6. Destroy Root EKS Cluster</h3> 
<pre><code class="language-bash">cd ../spire-root-cluster/
terraform destroy
</code></pre> 
<h3>7. Destroy Aurora MySQL</h3> 
<pre><code class="language-bash">cd ../../rds/spire-datastore/
terraform destroy
</code></pre> 
<h2>Cost Considerations</h2> 
<ul> 
 <li>EKS clusters: Pricing based on instance types and usage</li> 
 <li>Amazon Aurora MySQL: Serverless v2 pricing based on compute and storage</li> 
 <li>Data transfer: Costs for inter-cluster communication</li> 
</ul> 
<p>This blog post is based on Nested SPIRE deployment patterns and AWS EKS infrastructure. Always refer to the latest SPIRE documentation and AWS best practices for the most current deployment guidance.</p> 
<h2>Conclusion</h2> 
<p>This Nested SPIRE deployment provides a robust foundation for secure workload identity management across multiple Kubernetes clusters. The architecture scales well and provides the flexibility needed for complex microservices environments while maintaining strong security postures.</p> 
<p>By following this post, you’ve learned how to deploy SPIRE in a nested configuration on Amazon EKS, configure workload attestation and identity management, implement service-to-service authorization policies, monitor and troubleshoot your SPIFFE/SPIRE deployment, and apply best practices for production deployments.</p> 
<p>The combination of AWS managed services (Amazon EKS, Amazon Aurora) with the Nested SPIRE architecture creates a production-ready identity infrastructure that can support enterprise-scale applications with stringent security requirements.</p> 
<p>As cloud-native architecture continues to evolve, SPIFFE/SPIRE will play an increasingly important role in securing distributed systems. Consider exploring integrations with service meshes like Istio and monitoring tools like Prometheus to further enhance your security posture.</p> 
<p><strong>Ready to get started?</strong> Clone the <a href="https://github.com/aws-samples/sample-aws-eks-spiffe-spire-auth" rel="noopener" target="_blank">sample repository</a> and deploy your own nested SPIRE environment on Amazon EKS. For questions or to share your implementation experiences, join the <a href="https://slack.spiffe.io/" rel="noopener" target="_blank">SPIFFE community Slack</a> or engage with the AWS community on <a href="https://repost.aws/">re:Post</a>.</p> 
<p>For additional support and advanced configurations, refer to the <a href="https://spiffe.io/docs/" rel="noopener" target="_blank">SPIRE documentation</a> and the <a href="https://github.com/spiffe/helm-charts-hardened" rel="noopener" target="_blank">Helm charts repository</a>.</p> 
<hr style="width: 80%;" /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone size-full wp-image-20636" height="160" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/18/ericcav.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Eric Cavalcanti</h3> 
  <p><a href="https://www.linkedin.com/in/ericcavalcanti/" rel="noopener" target="_blank">Eric Cavalcanti</a> is a Delivery Consultant at Amazon Web Services, specializing in containers and cloud-native architectures. He works with customers across a wide range of domains, quickly adapting to each customer’s unique needs and helping them deliver practical, production-ready solutions on AWS.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone size-full wp-image-20635" height="160" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/18/ggundal.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Gaurav Gundal</h3> 
  <p><a href="https://www.linkedin.com/in/gauravgundal/" rel="noopener" target="_blank">Gaurav Gundal</a> is a DevOps consultant with AWS Professional Services, helping customers build solutions on the customer platform. When not building, designing, or developing solutions, Gaurav spends time with his family, plays guitar, and enjoys traveling to different places.</p> 
 </div> 
</footer>
