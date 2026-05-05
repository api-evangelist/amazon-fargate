---
title: "Session policies for Amazon EKS Pod Identity"
url: "https://aws.amazon.com/blogs/containers/session-policies-for-amazon-eks-pod-identity/"
date: "Tue, 24 Mar 2026 18:17:32 +0000"
author: "Aditya Potdar"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>Today, we’re announcing the new <strong>session policies</strong> capability for Amazon Elastic Kubernetes Service (Amazon EKS) Pod Identity. With this new feature, you can dynamically scope down <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> permissions for your <a href="https://kubernetes.io/" rel="noopener noreferrer" target="_blank">Kubernetes</a> pods without creating additional IAM roles. Session policies give you a flexible alternative to creating separate IAM roles for each permission variation by specifying an inline IAM policy during Pod Identity association creation. This helps you to manage permissions at scale without multiplying your role count. This means that your Kubernetes applications can now operate with precisely the permissions required, following the principle of least privilege, while avoiding reaching IAM role limits in large-scale deployments.</p> 
<p>In this post, we demonstrate how to use session policies to dynamically scope down IAM permissions for your Kubernetes pods without creating additional IAM roles, and discuss important considerations when adopting this feature. At re:Invent 2023, <a href="https://aws.amazon.com/eks/" rel="noopener noreferrer" target="_blank">Amazon EKS</a> introduced the <a href="https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html" rel="noopener noreferrer" target="_blank">EKS Pod Identity feature</a>. This feature helps users to configure Kubernetes applications running on Amazon EKS with fine-grained IAM permissions to access AWS resources such as <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> buckets and <a href="https://aws.amazon.com/dynamodb/" rel="noopener noreferrer" target="_blank">Amazon DynamoDB</a> tables. This feature addressed many of the existing challenges of <a href="https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html" rel="noopener noreferrer" target="_blank">IAM Roles for Service Accounts (IRSA)</a> by removing the need to set up OpenID Connect (OIDC) providers for EKS clusters, streamlining IAM trust policies, and streamlining the experience through Amazon EKS APIs. Furthermore, it introduced support for <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html" rel="noopener noreferrer" target="_blank">IAM role session tags</a>, so IAM administrators can author a single permissions policy that can work across roles by allowing access to AWS resources based on matching tags.</p> 
<p>Through nearly continuous user feedback, we learned that customers often face challenges when they need different permission levels for pods running the same application. Common scenarios include the following:</p> 
<ul> 
 <li><strong>Platform Engineering teams managing multi-tenant EKS clusters</strong> where different customer workloads need varying levels of access to the same AWS services. For example, a software as a service (SaaS) application where each pod operates against the tenant specific resources and data and must assume only that tenant’s specific roles.</li> 
 <li><strong>Teams running multiple environments</strong> (dev, test, staging) in the same cluster where pods need different permission scopes based on their environment without creating separate IAM roles for each.</li> 
 <li><strong>Data processing workloads</strong> where pods need access to different subsets of S3 buckets or DynamoDB tables based on their specific function, but creating individual IAM roles for each variation would exceed the 5,000 <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html" rel="noopener noreferrer" target="_blank">IAM roles per account quota</a>.</li> 
</ul> 
<p>Previously, you had to choose between creating separate IAM roles (hitting quota limits), granting overly broad permissions (security risk), or using session tags (not supported by all <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html#all_svcs" rel="noopener noreferrer" target="_blank">AWS services</a>) to handle these scenarios. To address these challenges and support fine-grained permission control, we’re launching <strong>session policies for EKS Pod Identity</strong>. You can use session policies to apply inline IAM policies when EKS Pod Identity assumes an IAM role on behalf of your pods. These policies create an intersection between the permissions granted by the IAM role and the session policy. This effectively <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#policies_session" rel="noopener noreferrer" target="_blank">restricts permissions</a> to only what’s explicitly allowed in both policies. This feature works for both same-account scenarios and cross-account access patterns using IAM role chaining. For in-depth policy evaluation logic, refer to IAM Policy evaluation for <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-basics.html" rel="noopener noreferrer" target="_blank">single account</a> and <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html" rel="noopener noreferrer" target="_blank">cross-account</a> scenarios.</p> 
<h2>What is changing in EKS Pod Identity APIs</h2> 
<p>The following APIs are updated to introduce new request and response elements to pass session policies:</p> 
<p><a href="https://docs.aws.amazon.com/eks/latest/APIReference/API_CreatePodIdentityAssociation.html" rel="noopener noreferrer" target="_blank"><strong>CreatePodIdentityAssociation</strong></a>: API call to create an EKS Pod Identity association between a service account in an EKS cluster and an IAM role with EKS Pod Identity.</p> 
<p><strong>Request parameters</strong>:</p> 
<ul> 
 <li><strong>clusterName</strong>: The name of the cluster in which to create the association.</li> 
 <li><strong>namespace</strong>: The name of the Kubernetes namespace inside the cluster in which to create the association.</li> 
 <li><strong>serviceAccount</strong>: The name of the Kubernetes service account inside the cluster with which to associate the IAM credentials.</li> 
 <li><strong>roleArn</strong>: The Amazon Resource Name (ARN) of the IAM role to associate with the service account.</li> 
 <li><strong>targetRoleArn</strong>: The ARN of a target IAM role in another AWS account. When specified, EKS Pod Identity performs IAM role chaining: it first assumes the <code>roleArn</code>, then uses those credentials to assume the <code>targetRoleArn</code>. When you specify a session policy, EKS Pod Identity applies it when assuming the target role, not the source role.</li> 
 <li><em><strong>policy (new):</strong></em> An IAM policy in JSON format that you want to use as an inline session policy. This parameter is optional. The resulting session’s permissions are the intersection of the role’s identity-based policy and the session policy. The plaintext policy can’t exceed 2,048 characters.</li> 
 <li><strong>disableSessionTags</strong>: Defaults to false (session tags enabled) when not specified. Session tags cannot be used in combination with session policies due to <a href="https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html" rel="noopener noreferrer" target="_blank">AWS Security Token Service (AWS STS)</a> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html#reference_iam-quotas-entity-length" rel="noopener noreferrer" target="_blank">policy size quota</a>.</li> 
</ul> 
<p><a href="https://docs.aws.amazon.com/eks/latest/APIReference/API_UpdatePodIdentityAssociation.html" rel="noopener noreferrer" target="_blank"><strong>UpdatePodIdentityAssociation</strong></a>: API to update an existing pod identity association. Use this to update the IAM role, target IAM role, session policy, or <code>disableSessionTags</code> attributes of the association.</p> 
<p><strong>Request parameters</strong>:</p> 
<ul> 
 <li><strong>clusterName</strong>: The name of the cluster where the association exists</li> 
 <li><strong>associationId</strong>: The ID of the association to be updated</li> 
 <li><strong>roleArn</strong>: The new IAM role to associate with the service account (optional)</li> 
 <li><strong>targetRoleArn</strong>: The new target IAM role to associate with the service account</li> 
 <li><strong>policy</strong> (new): The new session <strong>policy</strong> to associate with the service account (optional)</li> 
 <li><strong>disableSessionTags</strong>: Boolean flag to enable or disable session tags (optional, but must be <code>true</code> when the <strong>policy</strong> is specified)</li> 
</ul> 
<h2>How to get started</h2> 
<p>We demonstrate how to use session policies to restrict a Kubernetes pod’s permissions to only list S3 buckets, even though the underlying IAM role has broader S3 permissions including the ability to create buckets.</p> 
<h3>Prerequisites</h3> 
<p>The following prerequisites are necessary to complete this solution:</p> 
<ul> 
 <li>An AWS account</li> 
 <li>The latest version of <a href="https://aws.amazon.com/cli/" rel="noopener noreferrer" target="_blank">AWS Command Line Interface (AWS CLI)</a> configured on your device or <a href="https://docs.aws.amazon.com/cloudshell/latest/userguide/welcome.html#how-to-get-started" rel="noopener noreferrer" target="_blank">AWS CloudShell</a></li> 
 <li>CLI for Amazon EKS (<a href="https://eksctl.io/" rel="noopener noreferrer" target="_blank">eksctl</a>) for creating and managing EKS clusters</li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#kubectl-install-update" rel="noopener noreferrer" target="_blank">kubectl</a>, a CLI tool to interact with the Kubernetes API server</li> 
</ul> 
<h3>Setup</h3> 
<div class="hide-language"> 
 <pre><code class="lang-javascript">export&nbsp;AWS_REGION=us-west-2 &nbsp;# Replace with your AWS Region
export&nbsp;CLUSTER_NAME=session-policy-demo &nbsp;# Replace with your EKS cluster name
export&nbsp;NAMESPACE=demo-ns
export AWS_ACCOUNT=$(aws sts get-caller-identity --query Account --output text) # Replace with your AWS Account number
export&nbsp;SERVICE_ACCOUNT=s3-demo-sa</code></pre> 
</div> 
<h3>Step 1: Create an EKS cluster with Pod Identity add-on</h3> 
<p>Let’s start by creating an Amazon EKS Cluster using eksctl with the new eks-pod-identity-agent add-on.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">cat &lt;&lt; EOF &gt;&nbsp;cluster.yaml 
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
&nbsp;&nbsp;name:&nbsp;${CLUSTER_NAME}
&nbsp;&nbsp;region: ${AWS_REGION}
&nbsp;&nbsp;version: "1.35"

addons:
&nbsp;&nbsp;- name: vpc-cni
&nbsp;&nbsp;- name: coredns
&nbsp;&nbsp;- name: kube-proxy
  - name: eks-pod-identity-agent&nbsp;
&nbsp;&nbsp; &nbsp;
managedNodeGroups:
&nbsp;&nbsp;- name:&nbsp;${CLUSTER_NAME}-mng
&nbsp;&nbsp; &nbsp;privateNetworking: true
&nbsp;&nbsp; &nbsp;minSize: 1
&nbsp;&nbsp; &nbsp;desiredCapacity: 1
&nbsp;&nbsp; &nbsp;maxSize: 1
EOF

eksctl create cluster -f cluster.yaml</code></pre> 
</div> 
<p>It takes approximately 15 minutes to get the cluster infrastructure created, you can verify the status using the following command or&nbsp;<a href="https://console.aws.amazon.com/eks/home" rel="noopener noreferrer" target="_blank">Amazon EKS console</a>.</p> 
<p><code>aws eks describe-cluster --name ${CLUSTER_NAME} --region ${AWS_REGION} --query 'cluster.status'</code></p> 
<p>The output of this command should be&nbsp;<strong>ACTIVE</strong>.</p> 
<p>After the cluster creation is completed, confirm that the eks-pod-identity-agent add-on is running in the cluster.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">eksctl get addon --cluster ${CLUSTER_NAME} --region ${AWS_REGION} --name eks-pod-identity-agent -o json 

[
&nbsp;&nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"Name": "eks-pod-identity-agent",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"Version": "v1.3.10-eksbuild.2",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"NewerVersion": "",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"IAMRole": "",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"Status": "ACTIVE",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"ConfigurationValues": "",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"Issues": null
&nbsp;&nbsp; &nbsp;}
</code></pre> 
</div> 
<h3>Step 2: Create an IAM role with broad S3 permissions</h3> 
<p>Create an IAM role with permissions to perform multiple S3 operations, including listing and creating buckets.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">cat &lt;&lt; EOF &gt; role_trust_policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "pods.eks.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:TagSession"
            ]
        }
    ]
}
EOF

cat &lt;&lt; EOF &gt; role_permission_policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:CreateBucket",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "*"
        }
    ]
}
EOF

aws iam create-role --role-name session-policy-demo-role \
&nbsp;&nbsp; &nbsp;--assume-role-policy-document file://role_trust_policy.json
aws iam put-role-policy --role-name session-policy-demo-role \
&nbsp;&nbsp; &nbsp;--policy-name S3BroadAccess \
&nbsp;&nbsp; &nbsp;--policy-document file://role_permission_policy.json</code></pre> 
</div> 
<h3>Step 3: Create a Pod Identity association without a session policy</h3> 
<p>Create a Kubernetes namespace and service account, then associate them with the IAM role.</p> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">
kubectl create namespace ${NAMESPACE}
kubectl create serviceaccount ${SERVICE_ACCOUNT}&nbsp;-n ${NAMESPACE}

aws eks create-pod-identity-association \
&nbsp;&nbsp; &nbsp;--cluster-name ${CLUSTER_NAME}&nbsp;\
&nbsp;&nbsp; &nbsp;--namespace ${NAMESPACE}&nbsp;\
&nbsp;&nbsp; &nbsp;--service-account ${SERVICE_ACCOUNT}&nbsp;\
&nbsp;&nbsp; &nbsp;--role-arn arn:aws:iam::${AWS_ACCOUNT}:role/session-policy-demo-role \
&nbsp;&nbsp; &nbsp;--region ${AWS_REGION}</code></pre> 
</div> 
<h3>Step 4: Test the pod with full IAM role permissions</h3> 
<p>Deploy a test pod and verify it can perform multiple S3 operations.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">kubectl run s3-test --image=amazon/aws-cli:latest \
&nbsp;&nbsp; &nbsp;--namespace=${NAMESPACE} --rm -it --restart=Never \
&nbsp;&nbsp; &nbsp;--overrides='{"spec":{"serviceAccountName":"'${SERVICE_ACCOUNT}'"}}' \
&nbsp;&nbsp; &nbsp;-- s3 ls

kubectl run s3-test --image=amazon/aws-cli:latest \
&nbsp;&nbsp; &nbsp;--namespace=${NAMESPACE} --rm -it --restart=Never \
&nbsp;&nbsp; &nbsp;--overrides='{"spec":{"serviceAccountName":"'${SERVICE_ACCOUNT}'"}}' \
&nbsp;&nbsp; &nbsp;-- s3 mb s3://session-policy-test-bucket-$(date +%s)</code></pre> 
</div> 
<p>Both commands should succeed, demonstrating that the pod has full permissions granted by the IAM role.</p> 
<h3>Step 5: Add a session policy to restrict permissions</h3> 
<p>Now, update the Pod Identity association to include a session policy that only allows listing S3 buckets.</p> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">ASSOCIATION_ID=$(aws eks list-pod-identity-associations \
&nbsp;&nbsp; &nbsp;--cluster-name ${CLUSTER_NAME} \
&nbsp;&nbsp; &nbsp;--namespace ${NAMESPACE} \
&nbsp;&nbsp; &nbsp;--service-account ${SERVICE_ACCOUNT} \
&nbsp;&nbsp; &nbsp;--region ${AWS_REGION} \
&nbsp;&nbsp; &nbsp;--query 'associations[0].associationId' \
&nbsp;&nbsp; &nbsp;--output text)

aws eks update-pod-identity-association \
&nbsp;&nbsp; &nbsp;--cluster-name ${CLUSTER_NAME} \
&nbsp;&nbsp; &nbsp;--association-id ${ASSOCIATION_ID} \
&nbsp;&nbsp; &nbsp;--policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"s3:ListAllMyBuckets","Resource":"*"}]}' \
&nbsp;&nbsp; &nbsp;--disable-session-tags \
&nbsp;&nbsp; &nbsp;--region ${AWS_REGION}</code></pre> 
</div> 
<p><strong>Important</strong>: The <code>--disable-session-tags</code> flag is <strong>required</strong> when using session policies. Session tags and session policies cannot be used together due to packed policy size limitations enforced by AWS STS. If you attempt to create or update a pod identity association with both a policy and session tags enabled (or omit the <code>--disable-session-tags</code> flag), you will receive a <code>PackedPolicyTooLarge</code> validation error during the API call.</p> 
<h3>Step 6: Verify the restricted permissions</h3> 
<p>Due to the eventual consistency nature of the EKS Pod Identity API, it can take a few seconds to propagate the update. Run the following commands to test the scenario again.</p> 
<p><strong>Security Note:</strong> During the propagation window (up to 10 seconds), pods continue to receive credentials with previous permissions. Plan permission changes accordingly and monitor AWS CloudTrail for any unauthorized access attempts during the transition window.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">kubectl run s3-test --image=amazon/aws-cli:latest \
&nbsp;&nbsp; &nbsp;--namespace=${NAMESPACE} --rm -it --restart=Never \
&nbsp;&nbsp; &nbsp;--overrides='{"spec":{"serviceAccountName":"'${SERVICE_ACCOUNT}'"}}' \
&nbsp;&nbsp; &nbsp;-- s3 ls

kubectl run s3-test --image=amazon/aws-cli:latest \
&nbsp;&nbsp; &nbsp;--namespace=${NAMESPACE} --rm -it --restart=Never \
&nbsp;&nbsp; &nbsp;--overrides='{"spec":{"serviceAccountName":"'${SERVICE_ACCOUNT}'"}}' \
&nbsp;&nbsp; &nbsp;-- s3 mb s3://session-policy-test-bucket-2-$(date +%s)</code></pre> 
</div> 
<p>The first command succeeds because <code>s3:ListAllMyBuckets</code> is allowed by both the IAM role and the session policy. The second command fails with an “Access Denied” error because <code>s3:CreateBucket</code> is not included in the session policy, even though the IAM role permits it.</p> 
<h3>Step 7: Expand the session policy</h3> 
<p>You can update the session policy at any time to grant additional permissions (up to what the IAM role allows).</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">aws eks update-pod-identity-association \
&nbsp;&nbsp; &nbsp;--cluster-name ${CLUSTER_NAME}&nbsp;\
&nbsp;&nbsp; &nbsp;--association-id ${ASSOCIATION_ID}&nbsp;\
&nbsp;&nbsp; &nbsp;--policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["s3:ListAllMyBuckets","s3:CreateBucket"],"Resource":"*"}]}'&nbsp;\
&nbsp;&nbsp; &nbsp;--region ${AWS_REGION}</code></pre> 
</div> 
<p>After waiting for the credential cache to expire, the pod will now be able to both list and create S3 buckets. <a href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html" rel="noopener noreferrer" target="_blank">AWS CloudTrail</a> records all events, so you can audit and monitor all IAM role activity, including the session policies, for security and compliance purposes.</p> 
<h2>Understanding session policy validation</h2> 
<p>EKS Pod Identity validates session policies when you create or update a pod identity association, providing immediate feedback if there are any issues. The validation process includes:</p> 
<ol> 
 <li><strong>JSON format validation</strong>: EKS Pod Identity validates that the policy is valid JSON</li> 
 <li><strong>Character validation</strong>: EKS Pod Identity verifies that your policy contains only valid characters</li> 
 <li><strong>Size validation</strong>: EKS Pod Identity confirms that the policy doesn’t exceed 2,048 characters</li> 
 <li><strong>IAM policy schema validation</strong>: EKS Pod Identity validates against IAM policy structure requirements by performing a dry-run call to AWS STS</li> 
 <li><strong>Packed policy size validation</strong>: EKS Pod Identity makes sure that the session policy doesn’t exceed <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html#reference_iam-quotas-entity-length" rel="noopener noreferrer" target="_blank">STS limits</a> after compression</li> 
</ol> 
<h2>Important considerations</h2> 
<p>This section outlines key constraints and requirements before adopting session policies with EKS Pod Identity. It covers their interaction with session tags, permission evaluation rules, and how policies are applied in same and cross-account scenarios.</p> 
<h3>Session tags and session policies</h3> 
<p>Session tags can’t be used with session policies. When you specify a session policy, you must set <code>--disable-session-tags</code> to <code>true</code>. This is a requirement, not an optional optimization.</p> 
<h3>Permission intersection</h3> 
<p>Session policies can only restrict permissions, not expand them. The effective permissions are always the intersection of:</p> 
<ol> 
 <li>The IAM role’s identity-based policies</li> 
 <li>The session policy (if provided)</li> 
</ol> 
<p>This ensures that session policies maintain security boundaries and cannot be used to escalate privileges beyond what the IAM role allows.</p> 
<h3>Session policy application</h3> 
<p>Session policies are applied differently depending on whether you’re using a target role. A target role enables cross-account access scenarios where your pods must assume an IAM role in a different AWS account. When you specify a target role, EKS Pod Identity performs the IAM role chaining: it first assumes the source role (roleArn) in your account, then uses those credentials to assume the target role (targetRoleArn) in another account.</p> 
<p><strong>When </strong><code>targetRoleArn</code><strong> is specified (cross-account scenarios):</strong></p> 
<ul> 
 <li>The session policy is applied when assuming the <strong>target role</strong>, not the initial source role</li> 
 <li>The source role (<code>roleArn</code>) needs permission to assume the target role using <code>sts:AssumeRole</code> and <code>sts:TagSession</code> actions</li> 
 <li>The session policy restricts the permissions of the target role through IAM role chaining</li> 
</ul> 
<p><strong>When </strong><code>targetRoleArn</code><strong> is not specified (same-account scenarios):</strong></p> 
<ul> 
 <li>EKS Pod Identity applies the session policy directly to the roleArn when it assumes the role</li> 
 <li>The effective permissions are the intersection of the role’s identity-based policies and the session policy</li> 
</ul> 
<h3>Session policies design considerations</h3> 
<p>Session policies complement least-privilege IAM role design rather than replace it. The foundation remains designing IAM roles with the minimum necessary permissions for your workloads. Session policies become valuable when runtime context requires further scoping—such as in multi-tenant architectures or when a single role must serve varying permission requirements across different execution contexts. Use session policies where maintaining highly granular, context-specific IAM roles would create operational complexity without corresponding security benefit.</p> 
<h2>Other considerations</h2> 
<p>Following are additional considerations on service and AWS Region availability, Kubernetes version, and infrastructure as code (IaC) requirements.</p> 
<ul> 
 <li><strong>Kubernetes version support:</strong> The EKS Pod Identity session policy feature is supported on <a href="https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-release-calendar" rel="noopener noreferrer" target="_blank">Kubernetes versions</a> supported in Amazon EKS.</li> 
 <li><strong>Region availability:</strong> Session policies are available in AWS Commercial, Mainland China, and AWS GovCloud (US) Regions where Amazon EKS is supported.</li> 
 <li><strong>Role association limits:</strong> You can associate only one IAM role to a Kubernetes service account using EKS Pod Identity. However, you can update the role, target role, and session policy later.</li> 
 <li><strong>Tooling support:</strong> You can use AWS CloudFormation, eksctl, AWS CLI, and Amazon EKS API to create pod identity associations with session policies.</li> 
</ul> 
<h2>Cleaning up</h2> 
<p>Important: Complete these cleanup steps to avoid ongoing charges for the resources created in this walkthrough:</p> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">aws eks delete-pod-identity-association \
&nbsp;&nbsp; &nbsp;--cluster-name ${CLUSTER_NAME} \
&nbsp;&nbsp; &nbsp;--association-id ${ASSOCIATION_ID} \
&nbsp;&nbsp; &nbsp;--region ${AWS_REGION}

kubectl delete serviceaccount ${SERVICE_ACCOUNT} -n ${NAMESPACE}
kubectl delete namespace ${NAMESPACE}

eksctl delete cluster -f cluster.yaml

aws iam delete-role-policy --role-name session-policy-demo-role --policy-name S3BroadAccess
aws iam delete-role --role-name session-policy-demo-role</code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated the new <a href="https://docs.aws.amazon.com/eks/latest/userguide/pod-id-association.html#pod-id-association-create" rel="noopener noreferrer" target="_blank">session policy capability of Amazon EKS Pod Identity</a>, which enables fine-grained permission control for Kubernetes workloads without the overhead of managing thousands of IAM roles. With dynamic permission scoping at the Pod Identity level, this feature helps you build more secure, scalable, and maintainable applications on Amazon EKS while following least privilege. We encourage you to start using this feature and share your feedback at <a href="https://github.com/aws/containers-roadmap" rel="noopener noreferrer" target="_blank">AWS Containers Roadmap</a>.</p> 
<h2>Additional resources</h2> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html" rel="noopener noreferrer" target="_blank">Amazon EKS Pod Identity Documentation</a></li> 
 <li><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#policies_session" rel="noopener noreferrer" target="_blank">IAM session policies</a></li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/best-practices/security.html" rel="noopener noreferrer" target="_blank">AWS Security Best Practices for EKS</a></li> 
 <li><a href="https://aws.amazon.com/blogs/containers/amazon-eks-pod-identity-a-new-way-for-applications-on-eks-to-obtain-iam-credentials/" rel="noopener noreferrer" target="_blank">EKS Pod Identity: A new way for applications on EKS to obtain IAM credentials</a></li> 
</ul> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-20358 size-full alignleft" height="101" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/25/aditya.jpeg" width="100" /><strong>Aditya Potdar</strong> is a Software Engineer on the Amazon EKS team at Amazon Web Services. With experience spanning multiple leading technology companies, he specializes in architecting and delivering container-based infrastructure that powers large-scale production systems and machine learning workloads.</p> 
<p style="clear: both;"><img alt="" class="wp-image-19432 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/11/19/asramaa.jpg" width="100" /><strong>Ashok Srirama </strong>is a Principal Solutions Architect at Amazon Web Services, based in Washington Crossing, PA. He specializes in serverless applications, containers, and architecting distributed systems. When he’s not spending time with his family, he enjoys watching cricket, and driving his bimmer.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-19686 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/11/27/george.jpg" width="100" /><strong>George John</strong> is a Senior Product Manager for Amazon Elastic Kubernetes Service (EKS) at AWS, where he drives product strategy and innovation for one of the industry’s leading managed Kubernetes platforms. In his role, George works closely with customers, partners, and the broader cloud-native community to shape the future of container orchestration on AWS. When he is not building products, he loves to explore the Pacific Northwest with his family.</p>
