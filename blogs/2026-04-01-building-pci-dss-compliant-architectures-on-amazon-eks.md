---
title: "Building PCI DSS-Compliant Architectures on Amazon EKS"
url: "https://aws.amazon.com/blogs/containers/building-pci-dss-compliant-architectures-on-amazon-eks/"
date: "Wed, 01 Apr 2026 15:46:40 +0000"
author: "Piyush Mattoo"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>As cloud adoption accelerates, organizations operating in regulated environments face unique challenges implementing Kubernetes-based solutions.&nbsp;Financial services institutions (FSIs), healthcare providers, and other entities subject to stringent compliance requirements must navigate the intersection of cloud-native technologies and regulatory obligations.&nbsp;<a href="https://aws.amazon.com/eks/" rel="noopener noreferrer" target="_blank">Amazon Elastic Kubernetes Service</a> offers a powerful platform for containerized workloads, but its use in regulated environments requires careful consideration of security, data protection, and compliance controls.</p> 
<p>A critical challenge for organizations processing payment card data is implementing secure, compliant&nbsp;Kubernetes infrastructure in&nbsp;<a href="https://www.pcisecuritystandards.org/" rel="noopener noreferrer" target="_blank">Payment Card Industry Data security Standard (PCI DSS)</a> environments.&nbsp;A common question is whether shared tenancy infrastructure can meet PCI DSS requirements, or if dedicated hosts are necessary. This architectural decision affects organizations that process credit card data, leverage Kubernetes for their infrastructure,&nbsp;regardless of their node provisioning solution: <a href="https://docs.aws.amazon.com/eks/latest/best-practices/karpenter.html" rel="noopener noreferrer" target="_blank">Karpenter</a>, <a href="https://docs.aws.amazon.com/eks/latest/best-practices/cas.html" rel="noopener noreferrer" target="_blank">Cluster Autoscaler</a>, <a href="https://docs.aws.amazon.com/eks/latest/best-practices/automode.html" rel="noopener noreferrer" target="_blank">EKS Auto Mode</a>, or manual node management.</p> 
<p>Organizations must balance multiple factors: PCI DSS compliance risks, violations and associated penalties, cost implications of shared vs single-tenancy environments, the importance of payment card data security, and the long-term operational implications of their architectural decisions. Wrong choices can lead to financial penalties, unnecessary infrastructure costs, and complex and costly architectural changes.</p> 
<p>In this post, we explore key considerations, best practices, and architectural decisions hosting applications on EKS in shared tenancy environments while maintaining PCI DSS compliance.&nbsp;Please note this information is for reference purposes only and does not constitute legal or compliance advice—customers remain responsible for making their own independent assessment, and AWS products or services are provided ‘as is’ without warranties, representations, or conditions of any kind.</p> 
<h2>Understanding PCI DSS Compliance and Available AWS Resources</h2> 
<p>AWS supports PCI DSS compliance on EC2 shared tenancy environments, and numerous customers run PCI&nbsp;DSS compliant&nbsp;workloads using this model.&nbsp;AWS provides several resources for customers deploying PCI workloads on shared infrastructure, particularly in Kubernetes environments. This guidance becomes especially critical as organizations increasingly adopt dynamic node provisioning solutions while maintaining compliance and limiting scope.</p> 
<ul> 
 <li>The&nbsp;<a href="https://us-east-1.console.aws.amazon.com/artifact/v2/reports/details/report-HDnyY89Xu5CWmbG1" rel="noopener noreferrer" target="_blank">PCI DSS Attestation of Compliance (AOC) and Responsibility Summary</a>, available through Artifact, offers detailed insights into AWS’s compliance with the PCI DSS and provides details on the shared responsibility for PCI DSS requirements.</li> 
 <li><a href="https://d1.awsstatic.com/whitepapers/compliance/architecting-pci-dss-segmentation-scoping-aws.pdf" rel="noopener noreferrer" target="_blank">PCI DSS Segmentation Scoping with AWS</a> guide provides direction on properly segmenting PCI environments to limit scope and risk.</li> 
 <li>For Kubernetes-specific implementations, <a href="https://d1.awsstatic.com/whitepapers/architecting-amazon-eks-for-pci-dss-compliance.pdf" rel="noopener noreferrer" target="_blank">Architecting with EKS for PCI compliance</a>&nbsp;offer targeted guidance for regulated environments.</li> 
 <li>The&nbsp;<a href="https://us-east-1.console.aws.amazon.com/artifact/v2/reports/details/report-9RNhIQWolrSBhJxz" rel="noopener noreferrer" target="_blank">AWS SOC2 report </a>provides assurance to AWS customers that we securely manage our environment and sensitive data.</li> 
 <li><a href="https://d1.awsstatic.com/whitepapers/compliance/pci-dss-compliance-on-aws-v4-102023.pdf" rel="noopener noreferrer" target="_blank">PCI DSS v4.0 on AWS Compliance Guide</a>&nbsp;provides guidance on how to meet different PCI requirements with AWS services.</li> 
 <li><a href="https://d1.awsstatic.com/whitepapers/compliance/Architecting_Amazon_EKS_and_Bottlerocket_for_PCI_DSS_Compliance.pdf" rel="noopener noreferrer" target="_blank">Architecting Amazon EKS and Bottlerocket for PCI DSS Compliance</a>&nbsp;offers specific guidance for EKS Bottlerocket deployments in PCI DSS environments.</li> 
</ul> 
<h2>Node Provisioning Considerations for PCI DSS Environments</h2> 
<p>Before diving into controls, it’s essential to understand the relationship between node provisioning choices and compliance requirements.</p> 
<h3>Node Provisioning Options for EKS</h3> 
<p>Amazon EKS supports multiple approaches to node provisioning:</p> 
<ul> 
 <li><strong>Karpenter</strong>: Flexible, high-performance autoscaler provisioning right-sized compute resources in response to application load</li> 
 <li><strong>Cluster Autoscaler</strong>: Traditional Kubernetes autoscaling solution that works with AWS Auto Scaling Groups</li> 
 <li><strong>EKS Auto Mode</strong>: AWS-managed compute that automatically provisions and manages nodes with minimal configuration</li> 
 <li><strong>Manual Node Management</strong>: Direct EC2 instance management without autoscaling</li> 
</ul> 
<p>Your node provisioning choice affects node lifecycle management, scaling policies, instance selection, and operational complexity. However, it does not affect the fundamental PCI DSS compliance requirements around Pod security standards, Network policies, RBAC, Access Controls, Encryption, Logging, monitoring or Vulnerability management. Compliance depends on how you architect your EKS cluster and implement security controls at the workload and cluster level, not on which tool provisions your nodes.</p> 
<h3>Shared Tenancy vs. Dedicated Hosts</h3> 
<p>Single-tenant nodes via Dedicated Hosts provide an additional layer of physical isolation desired by some regulatory frameworks, but this level of separation is not required by PCI DSS. A well-designed shared tenancy environment with appropriate security controls can meet PCI DSS requirements while maintaining operational efficiency and cost-effectiveness.</p> 
<p>The remainder of this post focuses on implementing the appropriate security controls that support PCI DSS compliance regardless of your infrastructure choices.</p> 
<h2>Recommended Best Practices for PCI DSS on EKS</h2> 
<h3>1. Kubernetes Namespace Isolation</h3> 
<p>Kubernetes namespaces logically separate workloads within an EKS cluster. By default, all pods across all namespaces can communicate freely. Proper namespace isolation supports segmentation of Cardholder Data Environments (CDE) from non-CDE workloads by restricting pod-to-pod communication.</p> 
<p><strong>A. Namespace Design</strong></p> 
<ul> 
 <li>Use dedicated namespaces per application or tenant</li> 
 <li>Restrict cross-namespace access — prohibit shared service accounts or volumes across namespace boundaries</li> 
</ul> 
<p><strong>B. Authentication and Authorization</strong></p> 
<ul> 
 <li>Define Kubernetes roles aligned to specific job functions; bind via RoleBindings</li> 
 <li>Prefer namespace-scoped roles and role bindings over ClusterRoles/ClusterRoleBindings to minimize blast radius</li> 
 <li>Use <a href="https://github.com/liggitt/audit2rbac" rel="noopener noreferrer" target="_blank">audit2rbac</a> to generate RBAC policy files from Kubernetes audit logs</li> 
 <li>Regularly audit RBAC policies to enforce least privilege.</li> 
 <li>Associate Kubernetes RBAC with AWS IAM using access entries and AWS IAM Authenticator</li> 
</ul> 
<p><strong>C. Policy Enforcement</strong></p> 
<ul> 
 <li>Enforce policies using Gatekeeper, Kyverno, or Kubewarden</li> 
 <li>Configure AWS Organizations SCPs as additional permission boundaries where required</li> 
 <li>Implement comprehensive logging and monitoring for RBAC-related events</li> 
</ul> 
<p>Strong RBAC configurations supports PCI DSS Requirement 1.3.2 by restricting pod traffic. Kubernetes namespaces also act as a network security control with regards to Requirement 1.4.1, 1.4.2, and 1.4.4 by placing guardrails on the traffic authorized to and through pods in a cluster. This also supports Requirement 7.2.5 by enforcing least privileges and access for systems, applications, or processes within a cluster. Applying CPU and memory limits per namespace prevents resource exhaustion and ensure payment processing workloads have guaranteed capacity, supporting PCI DSS Requirement 2.2.1. Resource limits prevent denial-of-service attacks from compromised container and ensure critical payment processing workloads have guaranteed resources available. The separation between requests (guaranteed resources) and limits (maximum allowed resources) enables efficient resource utilization while maintaining performance isolation.</p> 
<h3>2. Pod Security Controls</h3> 
<p><a href="https://kubernetes.io/docs/concepts/security/pod-security-standards/" rel="noopener noreferrer" target="_blank">Pod Security Standards</a> (PSS) define three policy levels — <strong>Privileged</strong>, <strong>Baseline</strong>, and <strong>Restricted</strong> enforcing security-related best practices through security. Pod Security Admission (PSA) is the built-in Kubernetes admission controller that enforces PSS policies at the namespace level, supporting PCI DSS Requirement 2.2 (including sub-requirements 2.2.1, 2.2.3, and 2.2.6) for secure configuration hardening standards. Additionally, PSS supports Requirement 11.5.2 for change detection by preventing unauthorized cluster modifications.</p> 
<p><strong>A. Pod Security Standards &amp; Admission</strong></p> 
<p>PSA operates in three modes per namespace via namespace labels. The enforce mode rejects non-compliant pods at admission and serves as the primary enforcement gate.</p> 
<ul> 
 <li>Enforce restricted levels for PCI CDE workloads, consider baseline levels for non-CDE systems</li> 
 <li>Core security requirements: 
  <ul> 
   <li>No privileged containers</li> 
   <li>Non-Root users</li> 
   <li>Read-only root filesystem</li> 
   <li>Drop capabilities unless explicitly required</li> 
  </ul> </li> 
</ul> 
<p>The Kubernetes API server will reject any pod that doesn’t meet the specified security requirements before it is even scheduled. This provides a critical control point that supports non-compliant workloads cannot enter your PCI environment. The audit mode logs policy violations to the Kubernetes audit logs for compliance reporting, supporting PCI DSS Requirement 10, which mandates logging access attempts. The warn mode provides immediate feedback to developers when attempting to deploy non-compliant configurations, enabling rapid remediation without blocking legitimate operations.</p> 
<p><strong>B. SecurityContext Best Practices </strong></p> 
<p>As per PCI DSS Requirement 2.2, below are securityContext best practices:</p> 
<ul> 
 <li><code>runAsNonRoot: true</code>&nbsp;Creates a security boundary limiting the impact of any potential compromise</li> 
 <li><code>readOnlyRootFilesystem: true</code> Prevents runtime modifications to the container image and blocks attackers from writing malicious files or modifying binaries</li> 
 <li><code>seccompProfile: RuntimeDefault</code>&nbsp;Applies a security filter to system calls made by container processes. Blocks dangerous system calls that aren’t needed for normal application operation</li> 
 <li>&nbsp;<code>runAsUser: 10001</code> Explicitly sets a non-root UID to prevent containers from running as root, even if runAsNonRoot is not enforced by the runtime</li> 
 <li><code>allowPrivilegeEscalation: false</code>&nbsp;Blocks containers from gaining additional privileges beyond those granted at startup. This prevents exploitation of SUID binaries, kernel vulnerabilities, or container runtime bugs that might otherwise allow an attacker to escalate from a limited container user to root privileges</li> 
</ul> 
<p><strong>C. Restrict Host Resources</strong></p> 
<ul> 
 <li><strong>Disable hostPath:</strong> Prevents containers from mounting host filesystem directories, blocking access to sensitive host files</li> 
 <li><strong>Disable hostNetwork:</strong> Prevents containers from sharing the host’s network namespace, preserving network policy enforcement and CDE segmentation</li> 
 <li><strong>Disable hostPID:</strong> Prevents containers from accessing host processes, blocking lateral movement and privilege escalation</li> 
 <li><strong>Disable hostIPC:</strong> Prevents containers from accessing host inter-process communication, protecting shared memory from exposure</li> 
</ul> 
<p>Disabling these controls maintains strong container-to-host isolation and mitigates container escape vulnerabilities.</p> 
<p><strong>D. Monitoring and Auditing using Falco</strong></p> 
<p><a href="https://falco.org/" rel="noopener noreferrer" target="_blank">Falco</a> provides runtime detection of security policy violations by monitoring system behavior catching attacks that bypass static controls. For example, while PSS prevents deploying privileged containers, Falco can detect if a container gain privileged access at runtime through a vulnerability exploit, supporting PCI DSS Requirement 10.4. <a href="https://aws.amazon.com/blogs/security/continuous-runtime-security-monitoring-with-aws-security-hub-and-falco/" rel="noopener noreferrer" target="_blank"> Falco integrates with&nbsp;Amazon CloudWatch&nbsp;</a>to captures administrative actions (Requirement 10.2.1), and records events details (Requirement 10.2.2) . CloudWatch log access can be restricted to read-only for Requirement 10.3.1 and 10.3.2, and CloudWatch retention and storage policies satisfy Requirement 10.3.3 and 10.3.4.</p> 
<p>Amazon GuardDuty complements Falco by providing managed threat detection at the AWS level. GuardDuty EKS Runtime Monitoring deploys a fully managed EKS add-on that monitors container runtime activities — including file access, process execution, and network connections — to detect threats that bypass Kubernetes-layer controls. GuardDuty analyzes both Kubernetes audit logs and runtime events, detecting anomalous behaviors such as unexpected network connections, suspicious process execution, and unauthorized access attempts. This supports continuous monitoring requirements under PCI DSS Requirement 11.5.1 for intrusion detection.</p> 
<h3>3. Multi-Tenant Scheduling &amp; Node Isolation</h3> 
<p>Node isolation and workload scheduling controls are fundamental to PCI DSS compliance in EKS environments, regardless of node provisioning approach. The following practices support workload isolation and PCI DSS Requirements 2.2.3 and 2.2.6.</p> 
<p><strong>A. Taints and Tolerations</strong></p> 
<p>Configure <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/" rel="noopener noreferrer" target="_blank">node taints</a> per tenant group to restrict pod scheduling to specifically tainted nodes</p> 
<p><strong>B. Label Matching</strong></p> 
<p>Implement nodeSelector or affinity rules to limit workload deployment to appropriately labeled nodes.</p> 
<p><strong>C.&nbsp;Dedicated Node Groups per Tenant</strong></p> 
<p>Create dedicated node groups per tenant using your chosen provisioning method:</p> 
<ul> 
 <li><strong>Karpenter</strong>: Use dedicated NodePools per tenant with per-tenant EC2NodeClass</li> 
 <li><strong>Cluster Autoscaler</strong>: Configure separate Auto Scaling Groups per tenant</li> 
 <li><strong>EKS Auto Mode</strong>:&nbsp;Leverage automated compute management with appropriate taints and labels for tenant isolation. Auto Mode automatically provisions and manages compute resources while respecting your security boundaries through Kubernetes-native controls</li> 
 <li><strong>Manual Management</strong>: Create distinct node groups with tenant-specific configurations</li> 
</ul> 
<p><strong>D.&nbsp;Inter-pod Anti-affinity Configuration</strong></p> 
<p>Implement&nbsp;Hard Constraints (<code>requiredDuringSchedulingIgnoredDuringExecution</code>) to prevent service pods from different tenants from being co-scheduled on the same node.</p> 
<h3>4. Network Segmentation</h3> 
<p>Kubernetes network policies restrict network traffic between pods through Ingress and Egress rules, supporting PCI DSS Requirement 2.2 by establishing a secure hardening baseline at the cluster level.&nbsp;Additionally, they support Requirements 1.3.2, 1.4.1, 1.4.2, and 1.4.4 by implementing guardrails for authorized pod traffic.&nbsp;VPCs, AWS Network Firewall, AWS WAF, and Elastic Load Balancers can further enforce network segmentation at the infrastructure level.</p> 
<p>Amazon EKS now supports three <a href="https://aws.amazon.com/blogs/containers/amazon-eks-introduces-enhanced-network-policy-capabilities/" rel="noopener noreferrer" target="_blank">network policy tiers</a> evaluated in hierarchical order: <strong>Admin Tier → Standard NetworkPolicies </strong>→ <strong>Baseline Tier</strong>.</p> 
<p><strong>A. Default Deny Policy</strong></p> 
<p>Apply a <a href="https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-traffic" rel="noopener noreferrer" target="_blank">default deny-all policy</a> (no ingress or egress) to CDE namespaces before selectively permitting required traffic.</p> 
<p><strong>B. Enforce TLS for all communications for encryption in transit</strong></p> 
<p><strong>C. Admin Network Policies</strong></p> 
<p>Admin Network Policies ( ClusterNetworkPolicy, requires VPC CNI v1.21.1+, Kubernetes 1.29+) enable centralized, cluster-wide traffic control that cannot be overridden by namespace-level policies. For example: use the Admin Tier to enforce organization-wide CDE isolation and the Baseline Tier for default connectivity that developers can override.</p> 
<p><strong>D. Application Network Policies</strong></p> 
<p>Application Network Policies (EKS Auto Mode only) extend standard policies with DNS-based (FQDN) filtering at Layer 7, eliminating IP/CIDR management for external services — particularly valuable for payment gateway integrations and SaaS services with dynamic IPs.</p> 
<p>Additional network security controls include:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/containers/introducing-security-groups-for-pods/" rel="noopener noreferrer" target="_blank">Pod security groups</a> integration with EC2</li> 
 <li>AWS Network Firewall for CDE perimeter protection</li> 
 <li>VPC Flow Logs for comprehensive monitoring</li> 
 <li>AWS PrivateLink for secure AWS service communication</li> 
</ul> 
<h3>5. Storage Isolation</h3> 
<p>Proper storage isolation protects cardholder data and maintains PCI DSS compliance in multi-tenant EKS environments. Encrypted storage with fine-grained access controls ensures sensitive data remains protected at rest and in transit while preventing unauthorized cross-tenant access.</p> 
<p><strong>A. Volume Management</strong></p> 
<ul> 
 <li>Implement per-tenant <a href="https://aws.amazon.com/ebs/" rel="noopener noreferrer" target="_blank">EBS volumes</a> or <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/" rel="noopener noreferrer" target="_blank">PersistentVolumeClaim</a> (PVCs)</li> 
 <li>Enable encryption at rest using <a href="https://aws.amazon.com/kms/" rel="noopener noreferrer" target="_blank">AWS Key Management Service</a> (KMS)</li> 
 <li>Configure EBS volumes with encryption for EKS worker nodes</li> 
 <li>Use EBS CSI driver with encryption for persistent volumes</li> 
</ul> 
<p>It is important to note that EBS volume encryption with KMS alone does not satisfy PCI DSS Requirement 3.5.1.2 for PAN protection. Use the&nbsp;<a href="https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html" rel="noopener noreferrer" target="_blank">AWS Encryption SDK </a>to encrypt PAN before writing to EBS volumes to support this Requirement.</p> 
<p><strong>B. Security Controls</strong></p> 
<ul> 
 <li>Enable encryption in transit for storage drivers (<a href="https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html" rel="noopener noreferrer" target="_blank">EFS CSI</a>, <a href="https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html" rel="noopener noreferrer" target="_blank">EBS CSI</a>)</li> 
 <li>Restrict storage access using Roles and RoleBindings&nbsp;at the namespace level, limit service account permissions</li> 
 <li>Avoid hostPath or node-local storage</li> 
 <li>Use security contexts in pod specifications to storage access control</li> 
 <li>Implement encrypted backups</li> 
</ul> 
<h3>6. Fine grained IAM Isolation</h3> 
<p><a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management </a>(IAM) enables organizations to securely manage and scale access control for both workloads and workforce, serving as the foundation for implementing fine-grained policies and permissions within EKS environments. While customers retain responsibility for meeting PCI DSS Requirement 7, AWS IAM inherently supports Requirement 7.3.3 through its default “deny all” access control system. For comprehensive guidance on using AWS IAM to support Requirements 8 and 9, refer to&nbsp;<a href="https://d1.awsstatic.com/whitepapers/compliance/pci-dss-compliance-on-aws-v4-102023.pdf" rel="noopener noreferrer" target="_blank">PCI DSS v4.0 on AWS Compliance Guide</a>.</p> 
<p><strong>A. Identity and Role Management</strong></p> 
<ul> 
 <li>Implement IAM Roles for Service Accounts (IRSA) or&nbsp;<a href="https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html">Amazon EKS Pod Identity</a></li> 
 <li>Assign dedicated IAM roles per tenant namespace</li> 
 <li>Configure specific IAM roles for Kubernetes service accounts</li> 
</ul> 
<p><strong>B. Permission Scoping and Management</strong></p> 
<ul> 
 <li>Define granular IAM policies granting minimum required permissions</li> 
 <li>Avoid wildcard permissions (*) in policy statements</li> 
 <li>Conduct regular audits using <a href="https://aws.amazon.com/iam/access-analyzer/" rel="noopener noreferrer" target="_blank">AWS IAM Access Analyzer</a> to identify and remove unused permissions</li> 
 <li>Utilize <a href="https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html" rel="noopener noreferrer" target="_blank">AWS Security Token Service</a> (STS) for temporary credentials</li> 
</ul> 
<p><strong>C. Access Control Mechanisms</strong></p> 
<ul> 
 <li>Enable <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html" rel="noopener noreferrer" target="_blank">Multi-Factor Authentication</a> (MFA) for IAM users with console access</li> 
 <li>Implement access entries (Cluster identities linked to IAM principals) and policies (Authorization rules for specific cluster actions)</li> 
</ul> 
<p><strong>D. Monitoring and Compliance</strong></p> 
<ul> 
 <li>Configure <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html" rel="noopener noreferrer" target="_blank">CloudWatch alarms</a> to detect suspicious IAM activity</li> 
 <li>Deploy <a href="https://aws.amazon.com/config/" rel="noopener noreferrer" target="_blank">AWS Config</a> rules for policy violation detection</li> 
 <li>Implement Service Control Policies (SCPs) to enforce organizational compliance guardrails</li> 
</ul> 
<h3>7. Logging, Monitoring, and Audit Trails</h3> 
<p>Organizations can leverage multiple AWS services and tools to fulfill PCI DSS Requirement 10 logging mandates. A comprehensive logging and monitoring strategy combines AWS native services with container-specific monitoring tools to create a complete audit trail for compliance assessment and security monitoring.</p> 
<p><strong>A. AWS Native Services</strong></p> 
<ul> 
 <li><strong>AWS CloudTrail:</strong> Records EKS control plane actions performed and API activity logs for compliance auditing</li> 
 <li><strong>Amazon CloudWatch: </strong>Receives EKS Control Plane logs (must be explicitly enabled); Collects cluster metrics via <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html" rel="noopener noreferrer" target="_blank">CloudWatch agent&nbsp;</a>; enables <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html" rel="noopener noreferrer" target="_blank">Container Insights</a> for enhanced EKS observability</li> 
 <li><strong>VPC Flow Logs: </strong>Captures network traffic metadata for network activity auditing</li> 
</ul> 
<p><strong>B. Container-Specific Logging</strong></p> 
<ul> 
 <li><strong>Fluentbit</strong> : Run <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html" rel="noopener noreferrer" target="_blank">FluentBit</a> as a DaemonSet. Collects cluster-wide container logs and integrates with Container Insights</li> 
 <li><strong>Kubernetes Audit Logs: </strong>Records cluster-level activities</li> 
</ul> 
<p><strong>C. Security Monitoring</strong></p> 
<ul> 
 <li><a href="https://aws.amazon.com/guardduty/" rel="noopener noreferrer" target="_blank">AWS GuardDuty</a> threat detection for EKS runtime and audit log events</li> 
 <li><a href="https://aws.amazon.com/security-hub/" rel="noopener noreferrer" target="_blank">AWS Security Hub</a> for centralized security posture management</li> 
 <li><a href="https://aws.amazon.com/inspector/" rel="noopener noreferrer" target="_blank">Amazon Inspector</a> for vulnerability assessment and prioritization</li> 
 <li>Node lifecycle event tracking and container-level metrics collection</li> 
</ul> 
<p>This comprehensive logging architecture supports multiple Requirement PCI DSS requirements for activity audit logging, audit log details, and log retention and management.</p> 
<p>The integration of <a href="https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/" rel="noopener noreferrer" target="_blank">Kubernetes audit logs</a>, Gatekeeper violation logs, and Falco alerts creates a robust audit trail that demonstrates continuous compliance monitoring and provides necessary evidence for compliance assessments.</p> 
<h3>8. Vulnerability Management</h3> 
<p>PCI DSS Requirement 6.3.1 establishes specific criteria for vulnerability management: implement processes to identify security vulnerabilities, assign risk rankings to vulnerabilities. PCI DSS Requirement 6.3.3 requires that critical security patches/updates be installed within one month, and others installed in a risk-ranked time frame.</p> 
<p><strong>A. Container Image Security:</strong></p> 
<p>Continuous Vulnerability Scanning (supports Requirement 6.2.3 and 6.2.4): Implement a multi-layered scanning strategy combining native AWS tooling with third-party solutions:</p> 
<ul> 
 <li>Enable&nbsp;<a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html" rel="noopener noreferrer" target="_blank">Amazon Elastic Container Registry </a>(ECR) image scanning and Amazon Inspector for continuous assessment</li> 
 <li>Integrate third-party scanners (Trivy, Prisma Cloud, Aqua Security, Snyk) into CI/CD pipelines for shift-left detection</li> 
 <li>Configure automated alerting for new CVEs and establish vulnerability remediation workflows</li> 
</ul> 
<p>References to third-party tools and services (including but not limited to Falco, HashiCorp Vault, Trivy, Sysdig Secure, Prisma Cloud, Aqua Security, and Snyk) are provided for illustrative purposes only. AWS does not endorse or make any representations regarding third-party tools, and customers should perform their own independent evaluation before adopting any third-party solution.</p> 
<p><strong>B. Infrastructure Security:</strong></p> 
<p>System Hardening (supports PCI DSS Requirement 2.2): Deploy <a href="https://aws.amazon.com/bottlerocket/" rel="noopener noreferrer" target="_blank">Bottlerocket OS</a> as the node operating system. Its minimal attack surface, immutable infrastructure model and automated update management reduce the hardening burden while satisfying Requirement 2.2 controls.</p> 
<p><strong>C. Infrastructure Vulnerability Management</strong></p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html" rel="noopener noreferrer" target="_blank">AWS Systems Manager Patch Manager</a> for node patching</li> 
 <li><a href="https://aws.amazon.com/security-hub/" rel="noopener noreferrer" target="_blank">AWS Security Hub </a>for centralized security findings</li> 
 <li><a href="https://aws.amazon.com/config/" rel="noopener noreferrer" target="_blank">AWS Config</a> for configuration compliance monitoring</li> 
</ul> 
<p><strong>D. Runtime Security Controls:</strong></p> 
<p>For container Runtime Protection, deploy runtime monitoring across multiple layers:</p> 
<ul> 
 <li>Falco and Sysdig Secure for behavioral threat detection</li> 
 <li>GuardDuty Runtime Monitoring for AWS-native threat intelligence</li> 
 <li><a href="https://www.redhat.com/en/topics/linux/what-is-selinux" rel="noopener noreferrer" target="_blank">SELinux</a> security controls</li> 
</ul> 
<p><strong>E. Secrets and Image Provenance</strong></p> 
<ul> 
 <li><a href="https://www.hashicorp.com/en/products/vault" rel="noopener noreferrer" target="_blank">HashiCorp Vault</a> for secrets management</li> 
 <li>ImagePolicyWebhook and Kyverno for image provenance enforcement and policy management</li> 
</ul> 
<p><strong>F. Security Integration Practices</strong></p> 
<p>Treat containers as immutable artifacts—patch images and redeploy rather than modifying live containers. This approach, combined with CI/CD-integrated scanning, ensures vulnerabilities are caught early and the security posture remains consistent across environments.</p> 
<h3>Conclusion</h3> 
<p>Amazon EKS supports PCI DSS environments with shared tenancy when strong security controls enforced at both node and workload levels. Your choice of node provisioning—Karpenter, Cluster Autoscaler, EKS Auto Mode, or manual management—does not change the compliance requirements. Each approach can achieve the same security posture when properly configured. Single tenancy via Dedicated Hosts is optional in most cases—not required by PCI DSS itself, though may be mandated by internal policy or warranted due to specific architectural requirements. The critical success factor is defense-in-depth: namespace isolation, Pod Security Standards, network policies, IAM controls, and comprehensive monitoring.</p> 
<p><strong>Ready to get started?</strong> Contact your AWS account team or visit the <a href="https://aws.amazon.com/compliance/" rel="noopener noreferrer" target="_blank">AWS Compliance Center</a> to learn more about architecting compliant workloads on AWS.</p> 
<p><em>Do you run PCI workloads on EKS? What controls have helped you achieve compliance? Share your insights in the comments!</em></p> 
<hr /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone size-thumbnail wp-image-20371" height="150" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/26/image-cont-971-1-150x150.png" width="150" />
  </div> 
  <h3 class="lb-h4">Piyush Mattoo</h3> 
  <p><a href="https://www.linkedin.com/in/piyush-mattoo-aws/" rel="noopener" target="_blank">Piyush Mattoo</a> is a Solutions Architect at Amazon Web Services, where he specializes in enterprise-scale distributed systems and modern software architecture. He works closely with customers to design and implement cloud-native solutions using containers and AWS services.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone size-full wp-image-20437" height="536" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/27/Picture1.jpg" width="468" />
  </div> 
  <h3 class="lb-h4">Mridul Chopra</h3> 
  <p><a href="https://www.linkedin.com/in/mridul-chopra-831583118/" rel="noopener" target="_blank">Mridul Chopra</a> is a Containers Specialist Technical Account Manager at AWS, where she guides enterprise customers through their container modernization journeys with a focus on Amazon EKS and Kubernetes. She helps organizations design secure, scalable cloud-native solutions focussing on implementing best practices for modern cloud infrastructure.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone size-thumbnail wp-image-20372" height="150" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/26/image-cont-972-1-150x150.png" width="150" />
  </div> 
  <h3 class="lb-h4">Ted Tanner</h3> 
  <p><a href="https://www.linkedin.com/in/ted-tanner/" rel="noopener" target="_blank">Ted Tanner</a> is a Principal Assurance Consultant and PCI DSS QSA with AWS Security Assurance Services, and has 25 years of IT, security, and compliance experience. He leverages this to provide AWS customers with guidance on compliance and security in the cloud, and how to build and optimize their cloud compliance programs. He is author of two and co-author on 3 whitepapers and guides, and enjoys unblocking customers for migrating to the cloud.</p> 
 </div> 
</footer>
