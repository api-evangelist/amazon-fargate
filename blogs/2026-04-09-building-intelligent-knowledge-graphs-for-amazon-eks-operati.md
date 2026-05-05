---
title: "Building intelligent knowledge graphs for Amazon EKS operations using AWS DevOps Agent"
url: "https://aws.amazon.com/blogs/containers/building-intelligent-knowledge-graphs-for-amazon-eks-operations-using-aws-devops-agent/"
date: "Thu, 09 Apr 2026 18:26:10 +0000"
author: "Vikram Venkataraman"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>Modern observability has evolved significantly with the emergence of AIOps, transforming how organizations monitor and maintain their cloud infrastructure. Today’s intelligent agents can seamlessly integrate with monitoring tools, knowledge bases, and ticketing systems to triage issues and propose mitigation steps with unprecedented speed. Despite these advances, reducing Mean Time to Identify (MTTI) and Mean Time to Resolve (MTTR) in complex microservices architectures remains a challenge. During a recent conversation with a customer running a sophisticated AIOps platform for Kubernetes operations, they expressed a familiar concern: while their tooling was powerful, identifying the true root cause of incidents was still remarkably difficult. Pod-to-pod communication creates a constantly shifting network topology that’s challenging to map and understand without relying on third-party providers or eBPF profiling. This adds operational overhead and complexity to an already demanding troubleshooting process.</p> 
<p>This is where AWS DevOps Agent changes the game. It goes beyond collecting insights from telemetry signals to build intelligent knowledge graphs that map the intricate relationships between your Amazon Elastic Kubernetes Service (Amazon EKS) resources. AWS DevOps Agent acts as your always-on DevOps engineer, autonomously investigating incidents and identifying operational improvements by learning your resources and their relationships. It works with your existing observability tools, runbooks, code repositories, and continuous integration and delivery (CI/CD) pipelines, correlating telemetry, code, and deployment data to understand the true topology of your applications—whether they run in the cloud or hybrid environments. For Amazon EKS specifically, the agent goes beyond cluster-level visibility, developing a deep understanding of Kubernetes objects and their interdependencies, from Services to Pods. This enables it to traverse dependency chains and pinpoint the deepest impaired object that’s likely causing your incident.</p> 
<p>In this post, we demonstrate how AWS DevOps Agent works—from alert generation to identifying the affected EKS cluster, building knowledge graphs, and troubleshooting application or infrastructure issues, ultimately reducing MTTI and MTTR for your Kubernetes operations.</p> 
<h2>Prerequisites</h2> 
<p>Complete the following prerequisites to continue with this post.</p> 
<ul> 
 <li>The <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI) version 2. For installation instructions, see <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener noreferrer" target="_blank">Installing or updating to the latest version of the AWS CLI</a>.</li> 
 <li><a href="https://helm.sh/docs/intro/install/" rel="noopener noreferrer" target="_blank">helm</a></li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html" rel="noopener noreferrer" target="_blank">Kubectl</a></li> 
 <li>An <a href="https://aws-ia.github.io/terraform-aws-eks-blueprints/getting-started/" rel="noopener noreferrer" target="_blank">EKS cluster</a>&nbsp;with Control plane logs enabled</li> 
 <li>Install&nbsp;<a href="https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html" rel="noopener noreferrer" target="_blank">Load Balancer Controller</a></li> 
 <li>AWS DevOps Agent Agentspace. For installation instructions, refer to <a href="https://docs.aws.amazon.com/devopsagent/latest/userguide/getting-started-with-aws-devops-agent-creating-an-agent-space.html" rel="noopener noreferrer" target="_blank">Creating an Agent Space</a></li> 
</ul> 
<h2>Deploy a sample retail application</h2> 
<p>For the post, we use <a href="https://github.com/aws-containers/retail-store-sample-app" rel="noopener noreferrer" target="_blank">Containers Retail Store Sample Application</a>. This is a purpose-built microservices application designed to demonstrate modern cloud architectures&nbsp;and container orchestration patterns. This application simulates a fully functional ecommerce platform with distributed&nbsp;components that showcase real-world operational challenges.&nbsp;The application consists of five microservices: <strong>UI Service, Catalog Service, Cart Service, Orders Service, Checkout Service</strong>. Each microservice is built with different technology stacks to represent heterogeneous production environments.</p> 
<p><img alt="Microservices architecture diagram showing a UI/Frontend service connecting to three backend services: Checkout, Cart, and Catalog" class="aligncenter size-full wp-image-20544" height="271" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1791.jpg" width="635" /></p> 
<p>Figure 1. Components of the sample application.</p> 
<p>Let’s go ahead and deploy this sample application in the EKS cluster that you have provisioned already:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">kubectl apply -f https://github.com/aws-containers/retail-store-sample-app/releases/latest/download/kubernetes.yaml
kubectl wait --for=condition=available deployments --all&nbsp;--timeout=120s
kubectl annotate svc ui service.beta.kubernetes.io/aws-load-balancer-scheme=internet-facing --overwrite</code></pre> 
</div> 
<h3>Enabling AWS DevOps Agent access for Amazon EKS cluster</h3> 
<p>Now that we have the sample application deployed, let’s integrate this cluster with AWS DevOps Agent to do troubleshooting. You can enable AWS DevOps Agent to describe your Kubernetes cluster objects, retrieve pod logs and cluster events, for Amazon EKS clusters (only accessible with a VPC).The Agent Space must have access to the EKS cluster. To provide access, we must get the role of the Agent Space and use that role in the EKS console to add an access entry to the EKS cluster.</p> 
<p>From the <strong>Agent Spaces</strong>, select the <strong>Agent Space</strong> that needs access to the Amazon EKS cluster and choose the <strong>View Details</strong> button to open the details of the Agent Space.</p> 
<p>Open the <strong>Capabilities</strong> tab, and under the <strong>Cloud</strong>&nbsp;section, select the <strong>primary source</strong> and choose <strong>Edit</strong>. This will open the primary account source and note down the role shown in the <strong>Role Name</strong> field. This is the role that needs access to the Amazon EKS cluster.</p> 
<p><img alt="Setting up EKS access entry" class="aligncenter size-full wp-image-20545" height="1130" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1792.jpg" style="margin: 10px 0px 10px 0px;" width="2004" /></p> 
<p>On the <strong>EKS console</strong>, select the cluster that you need to provide access to for the AWS DevOps agent and open the <strong>Access</strong> tab.</p> 
<p>Under the <strong>IAM Access Entries</strong> list, choose the <strong>create</strong> button to create a new Access entry.</p> 
<p>For the <strong>IAM Principal ARN</strong>, select the role from the Agent Space that was noted down from the previous section and choose <strong>Next</strong>.</p> 
<p>Under <strong>Access Policies,</strong> select <strong>AmazonAIOpsAssistantPolicy&nbsp;</strong>and provide the access scope as Cluster. Then choose the <strong>Add Policy</strong> button to add the selected policy and choose the <strong>Next</strong> button.</p> 
<p>The Review and Create screen will show the following details. Select the&nbsp;<strong>Create</strong> button to add the access entry.</p> 
<p><img alt="Add the access entry" class="aligncenter size-full wp-image-20546" height="860" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1793.jpg" width="1364" /></p> 
<p>This completes the EKS cluster setup and this EKS entry provides the DevOps agent access to the cluster. In the environment where you have multiple clusters, you can use CLI, Terraform, or GitOps to create the access entries in the clusters.</p> 
<p>After the access entry is added, the Kubernetes objects will be available for DevOps agent Topology Sources.</p> 
<p><img alt="Topology graph in DevOps Agent showing Kubernetes Objects" class="aligncenter size-full wp-image-20547" height="274" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1794.jpg" width="2268" /></p> 
<p>In addition to the overview of resources discovered, you can also see the service map diagram of various Kubernetes objects interacting across namespaces using the <strong>Learned Topology</strong> feature of AWS DevOps agent.</p> 
<p>Learned topology is an automatically generated knowledge graph that maps entities and relationships in your application environment through resource discovery, relationship detection, code/deployment mapping, and observability behavior mapping, continuously evolving as the agent completes more tasks.</p> 
<p>For visualizing EKS objects, follow the below steps:</p> 
<p>1. Navigate to your Agent Space’s Operator access console and click the Topology tab.</p> 
<p>2. Select your preferred view filter: <strong>Learned</strong></p> 
<p>3. Explore the interactive knowledge graph where nodes represents Kubernetes objects, lines show connections.</p> 
<p><img alt="Kubernetes Objects Knowledge graph" class="aligncenter size-full wp-image-20574" height="1534" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/09/Screenshot-2026-04-09-at-12.03.29 PM.png" width="3706" /></p> 
<p>Now that we’ve covered how the DevOps Agent integrates with Amazon EKS and the powerful capabilities that it brings to cluster operations, let’s explore how this integration solves real-world challenges that platform teams face daily.</p> 
<h4>Scenario 1 – Troubleshoot&nbsp;Kubernetes application availability issue with DevOps Agent</h4> 
<p>In this scenario, we demonstrate how AWS DevOps Agent autonomously investigates a Kubernetes application availability issue. You will see how the agent:</p> 
<ul> 
 <li><strong>Automatically triggers investigations</strong> when external health checks detect failures</li> 
 <li><strong>Builds a topology graph</strong> mapping the relationships between Amazon Route 53, Network Load Balancer, Kubernetes Services, and Pods</li> 
 <li><strong>Correlates multi-layer telemetry</strong> across AWS infrastructure metrics, Kubernetes events, and container logs</li> 
 <li><strong>Traverses dependency chains</strong> from the external endpoint down to the specific failing pod</li> 
 <li><strong>Identifies root causes</strong> by analyzing pod status, container logs, and recent deployment changes</li> 
 <li><strong>Generates actionable mitigation plans</strong> with specific remediation steps</li> 
</ul> 
<p>To experience this automated troubleshooting workflow, we set up a simulation environment. On successful setup, the environment will have the following components:</p> 
<ol> 
 <li>Route 53 health check continuously monitors the UI Network Load Balancer endpoint (HTTP/80) every 30 seconds</li> 
 <li>Amazon CloudWatch Alarm – <code>retail-store-ui-endpoint-down</code> triggers when health check fails for two consecutive periods</li> 
 <li>AWS Lambda function processes the alarm, generates an HMAC-signed webhook payload, and invokes the DevOps Agent</li> 
 <li>DevOps Agent receives the webhook, initiates an investigation, and queries the EKS API, Kubernetes API, CloudWatch Logs, and CloudWatch Metrics</li> 
</ol> 
<p><img alt="Troubleshooting workflow using AWS DevOps Agent" class="aligncenter size-full wp-image-20548" height="500" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1795.jpg" width="965" /></p> 
<p>Let’s now deploy the environment. To trigger DevOps Agent investigations automatically, we use the lambda function to invoke the agent’s webhook. To fetch the webhook, complete the following steps:</p> 
<h3>Step 1: Getting DevOps Agent’s webhook information</h3> 
<ol> 
 <li>Navigate to your <strong>Agent Space </strong>in the AWS DevOps Agent console.</li> 
 <li>Go to the <strong>Capabilities</strong> tab.</li> 
 <li>Under the <strong>Webhook</strong> section, choose <strong>Configure.</strong></li> 
 <li>Choose <strong>Generate webhook</strong> to create HMAC credentials.</li> 
 <li><strong>Save the webhook URL and secret</strong>. You will need these for the next step.</li> 
</ol> 
<h3>Step 2: Deploy</h3> 
<p>Extract the tar ball, configure the environment variables, and run the deploy script to create all the required resources.</p> 
<div class="hide-language"> 
 <pre><code class="lang-javascript">git clone&nbsp;https://github.com/aws-samples/Amazon-prometheus-bedrock-agent-example.git
cd Amazon-prometheus-bedrock-agent-example/devops-agent
tar -xzf scenario-1-deployment.tar.gz
cd scenario-1-deployment

export PRIMARY_REGION="us-east-1" &nbsp; &nbsp; &nbsp; &nbsp; 
export ENVIRONMENT_NAME="retail-store"

export&nbsp;WEBHOOK_URL="https://event-ai.us-east-1.api.aws/webhook/generic/YOUR-ID"
export&nbsp;WEBHOOK_SECRET="YOUR-SECRET-KEY"

chmod&nbsp;+x&nbsp;deploy.sh
./deploy.sh
</code></pre> 
</div> 
<h4>Step 3: Trigger a test investigation</h4> 
<p>To validate the end-to-end flow without waiting for a real failure, manually scale down the UI application replicas to 0 to trigger an alarm: <code>kubectl scale deployment ui --replicas=0</code>Within minutes, you should see a new investigation appear in your DevOps Agent Space web app.</p> 
<p>You can access&nbsp;the <strong>AWS DevOps Agent Operator Web App</strong> by completing the following steps:</p> 
<ol> 
 <li>Navigate to the AWS DevOps Agent Console.</li> 
 <li>Select your specific <strong>Agent Space</strong> from the list.</li> 
 <li>On the Agent Space landing page, go to the <strong>Web app</strong> tab.</li> 
 <li>Choose the&nbsp;<strong>Operator access.</strong></li> 
</ol> 
<p><img alt="Starting the DevOps Agent investigation" class="aligncenter size-full wp-image-20549" height="327" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1796.jpg" width="2560" /></p> 
<p><img alt="DevOps agent fetching relevant telemetry data" class="aligncenter size-full wp-image-20550" height="381" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1797.jpg" width="2560" /></p> 
<p><img alt="Correlating telemetry data" class="aligncenter size-full wp-image-20551" height="431" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1798.jpg" width="2560" /></p> 
<p><img alt="DevOps Agent identifying root cause" class="aligncenter size-full wp-image-20552" height="370" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-1799.jpg" width="2560" /></p> 
<p><img alt="Validating if the root cause is correct" class="aligncenter size-full wp-image-20553" height="401" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17910.jpg" width="2560" /></p> 
<p><img alt="DevOps Agent fetching the Kubernetes objects" class="aligncenter size-full wp-image-20554" height="1767" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17911.jpg" width="2560" /></p> 
<p><img alt="Arriving at the final root cause" class="aligncenter size-full wp-image-20555" height="1134" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17912.jpg" width="2560" /></p> 
<p><img alt="DevOps agent sharing the final root cause" class="aligncenter size-full wp-image-20556" height="394" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17913.jpg" width="1662" /></p> 
<p>AWS DevOps agent when configured with the right access can leverage kubectl commands to discover and fetch information from your Amazon EKS cluster.</p> 
<h4>Scenario 2 – Kubernetes Infrastructure and application dependencies troubleshooting</h4> 
<p>Application failures don’t always originate from your workloads. In production Kubernetes environments, critical cluster add-ons like CoreDNS, kube-proxy, and the Amazon Virtual Private Cloud (Amazon VPC) Container Network Interface plugin form the foundation of cluster operations. When these components experience issues, the symptoms can manifest across seemingly unrelated applications, making root cause identification challenging. In this scenario, we demonstrate how AWS DevOps Agent automatically correlates application-level symptoms with underlying infrastructure issues, significantly reducing the time required to identify and resolve failures in critical Kubernetes add-ons.</p> 
<p>We intentionally scale down the coredns replica:</p> 
<p><code>kubectl scale deployment coredns --replicas=0</code></p> 
<p>Let’s initiate an investigation:</p> 
<p><img alt="Scenario 2 investigation start" class="aligncenter size-full wp-image-20557" height="362" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17914.jpg" width="1878" /></p> 
<p>AWS DevOps Agent will go through your kube-events and pod logs of the kubernetes objects to identify the root cause. Within minutes, you should see the root cause of the down alerts:</p> 
<p><img alt="Root cause of scenario 2 shared by AWS DevOps Agent" class="aligncenter size-full wp-image-20558" height="574" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/containers-17915.jpg" width="1890" /></p> 
<p>You can provide additional context and troubleshooting guidance to the DevOps agent by adding a runbook in the <strong>Skills</strong>&nbsp;tab. A detailed EKS troubleshooting document is provided in the GitHub <a href="https://github.com/aws-samples/Amazon-prometheus-bedrock-agent-example/blob/main/docs/EKS%20Troubleshooting.docx" rel="noopener noreferrer" target="_blank">repo</a>.</p> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how AWS DevOps Agent transforms Amazon EKS operations by building intelligent knowledge graphs that map the complex relationships between your Kubernetes resources. By automatically correlating telemetry signals across infrastructure, application, and container layers, the agent significantly reduces MTTI and MTTR for incidents in your EKS environments.</p> 
<p>The power of AWS DevOps Agent lies in its ability to understand context, not only collect data. Instead of manually correlating logs, metrics, and events, the agent autonomously traces dependency chains—from external endpoints through load balancers, services, and pods—to pinpoint the exact source of failures. Whether troubleshooting application-level issues or critical infrastructure components like CoreDNS, the agent’s knowledge graph approach removes the guesswork that typically extends incident resolution times.</p> 
<p>As Kubernetes environments continue to grow in complexity with thousands of nodes and intricate microservices architectures, the need for intelligent, autonomous operations becomes critical. AWS DevOps Agent doesn’t only alert you to problems—it investigates them, understands their context within your broader infrastructure, and provides actionable remediation steps, acting as your always-on DevOps engineer.</p> 
<h2>Further reading</h2> 
<p>To learn more about AWS DevOps Agent, refer to the following resources:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/devops/from-ai-agent-prototype-to-product-lessons-from-building-aws-devops-agent/" rel="noopener noreferrer" target="_blank">From AI agent prototype to product: Lessons from building AWS DevOps Agent</a></li> 
 <li><a href="https://aws.amazon.com/blogs/devops/best-practices-for-deploying-aws-devops-agent-in-production/" rel="noopener noreferrer" target="_blank">Best Practices for Deploying AWS DevOps Agent in Production</a></li> 
 <li><a href="https://aws.amazon.com/blogs/mt/resolve-application-issues-autonomously-with-aws-devops-agent-and-dynatrace/" rel="noopener noreferrer" target="_blank">Resolve application issues autonomously with AWS DevOps Agent and Dynatrace</a></li> 
 <li><a href="https://catalog.us-east-1.prod.workshops.aws/workshops/767d3081-b4fa-4e08-81da-17e5c94a1a08" rel="noopener noreferrer" target="_blank">AWS DevOps Agent workshop</a></li> 
</ul> 
<hr /> 
<h3>About the authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-20560" height="151" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/vikram-1.png" width="100" />
  </div> 
  <h3 class="lb-h4">Vikram Venkataraman</h3> 
  <p>Vikram Venkataraman is a Principal Specialist Solutions Architect at Amazon Web Services (AWS). He helps customers modernize, scale, and adopt best practices for their containerized workloads. With the emergence of Generative AI, Vikram has been actively working with customers to leverage AWS’s AI/ML services to solve complex operational challenges, streamline monitoring workflows, and enhance incident response through intelligent automation.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-20561" height="160" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/Shiv-Rajendran.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Shivkumar</h3> 
  <p>Shivkumar is a Technical Account Manager at Amazon Web Services. He serves as a trusted advisor and advocate for AWS customers, proactively providing technical guidance and architectural best practices to help customers build, operate, and optimize solutions on AWS that achieve their business goals. He partners closely with customers across operations, development, and leadership to understand their needs and ensure they are leveraging AWS services effectively.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-20562" height="768" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/geppel.jpg" width="576" />
  </div> 
  <h3 class="lb-h4">Greg Eppel</h3> 
  <p>Greg Eppel is a Principal Specialist for DevOps Agent and has spent the last several years focused on Cloud Operations and helping AWS customers on their cloud journey.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-20563" height="99" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/08/sudheer-1.png" width="100" />
  </div> 
  <h3 class="lb-h4">Sudheer Sangunni</h3> 
  <p>Sudheer Sangunni is a Senior Technical Account Manager at AWS Enterprise Support. With his extensive expertise in the AWS Cloud and big data, Sudheer plays a pivotal role in assisting customers with enhancing their monitoring and observability capabilities within AWS offerings.</p> 
 </div> 
</footer>
