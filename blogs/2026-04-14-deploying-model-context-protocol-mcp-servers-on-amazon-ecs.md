---
title: "Deploying Model Context Protocol (MCP) servers on Amazon ECS"
url: "https://aws.amazon.com/blogs/containers/deploying-model-context-protocol-mcp-servers-on-amazon-ecs/"
date: "Tue, 14 Apr 2026 16:55:37 +0000"
author: "Sudheer Manubolu"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>Organizations are increasingly adopting AI agents to automate workflows across their operations. Common use cases include customer support, software development, business intelligence, supply chain management, and more. To be useful, these agents need access to internal tools, data sources, and business logic. Custom <a href="https://modelcontextprotocol.io/docs/getting-started/intro" rel="noopener noreferrer" target="_blank">Model Context Protocol (MCP)</a> servers have become one of the most popular ways to connect AI agents to tools and contexts. As organizations move from prototypes to production, hosting MCP servers on a reliable, scalable, and secure platform becomes critical.</p> 
<p>There are several options for hosting MCP servers on AWS, and each has its own strengths. <a href="https://docs.aws.amazon.com/lambda/latest/dg/welcome.html" rel="noopener noreferrer" target="_blank">AWS Lambda</a> works well for lightweight, stateless tool endpoints with bursty traffic patterns. <a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-mcp.html" rel="noopener noreferrer" target="_blank">Amazon Bedrock AgentCore </a>provides a fully managed agent orchestration suite with built-in identity, memory, and tool discovery. If your workloads need more control over the runtime, networking, and connection lifecycle, then <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html" rel="noopener noreferrer" target="_blank">Amazon Elastic Container Service (Amazon ECS)</a> on <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html" rel="noopener noreferrer" target="_blank">AWS Fargate </a>is a great option, which we will cover in this post. Amazon ECS lets you run your MCP server as a long-lived service with warm caches, persistent streaming connections, sidecars, and any language or runtime you choose. Amazon ECS integrates naturally with enterprise perimeter controls such as Application Load Balancers, AWS WAF, private subnets, and VPC endpoints. This makes Amazon ECS a strong fit for sessionful or streaming heavy servers, dependency heavy workloads bundling native libraries or large processing pipelines, high-throughput tool gateways that benefit from stable concurrency, and network-embedded servers that must reside inside a VPC alongside private data stores.</p> 
<p>In this post, we will walk you through a three-tier MCP application deployed entirely on Amazon ECS, using <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html" rel="noopener noreferrer" target="_blank">Service Connect</a> for service-to-service communication and <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/express-service-overview.html" rel="noopener noreferrer" target="_blank">Express Mode</a> for automated load balancing, to show how to take an MCP-based workload from concept to production.</p> 
<h2>Architecture</h2> 
<p>We consider a containerized MCP application consisting of three services: a Gradio-based UI, an AI Agent powered by Amazon Bedrock, and a FastMCP server, all running on Amazon ECS with AWS Fargate. Figure 1 illustrates the architecture.</p> 
<p><img alt="Architecture diagram showing Amazon VPC with public and private subnets, UI Service with ALB and Gradio UI, Agent Service with Strands Agent, and MCP Server on Amazon ECS connecting to Bedrock Nova Lite and S3 Product Catalog" class="alignnone size-full wp-image-20481" height="556" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/07/CONTAINERS-153-1.png" width="1430" /></p> 
<p style="text-align: center;"><strong>Figure 1: Three-tier MCP application on Amazon ECS</strong></p> 
<p>Here are the high-level steps in the request flow:</p> 
<ol> 
 <li>A user submits a natural language query (for example, “Find laptops under $1000”) through the Gradio UI, exposed to the internet via an Application Load Balancer provisioned by Amazon ECS Express Mode.</li> 
 <li>The UI forwards the request over Amazon ECS Service Connect to the Agent Service running in a private subnet.</li> 
 <li>The Agent invokes Amazon Bedrock (Amazon Nova 2 Lite model) to interpret the query and determine which MCP tools to call.</li> 
 <li>The Agent connects to the MCP Server over Amazon ECS Service Connect using the Streamable HTTP transport. Amazon ECS Service Connect handles the underlying network routing.</li> 
 <li>The MCP Server executes the tool call like searching, filtering, or retrieving product data from an Amazon Simple Storage Service (Amazon S3) bucket.</li> 
 <li>The MCP Server returns results through the chain: MCP Server → Agent → UI → User.</li> 
</ol> 
<p>All inter-service communication stays within the Amazon Virtual Private Cloud (Amazon VPC). Only the UI service is publicly accessible. The key components of this architecture are:</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Component</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Technology</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Role</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>UI Service</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Gradio on Amazon ECS</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Web chat interface, public-facing via ALB</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Agent Service</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Strands Agents SDK + Amazon Bedrock</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Orchestrates AI reasoning and MCP tool calls</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>MCP Server</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">FastMCP on Amazon ECS</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Exposes product catalog as MCP tools over Streamable HTTP</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Service Connect</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">AWS Cloud Map + Envoy proxy</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Service-to-service discovery and routing</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>ECS Express Mode</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Managed ALB + auto-scaling</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Automated public endpoint with HTTPS</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Product Catalog</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon S3</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Stores product data as JSON</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Infrastructure</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">AWS CloudFormation</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon VPC, subnets, IAM roles, Amazon ECS cluster</td> 
  </tr> 
 </tbody> 
</table> 
<p>The MCP Server uses the Streamable HTTP transport, which operates in stateless mode. Each tool call is a self-contained HTTP request with no server-side session state. This allows the MCP Server to scale horizontally across multiple replicas without session affinity. For workloads that require long-lived sessions, such as multi-step workflows with server-side context, Streamable HTTP also supports a stateful mode using the Mcp-Session-Id header, see the <a href="https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#streamable-http)" rel="noopener noreferrer" target="_blank">MCP specification</a> for details.</p> 
<h2>Walkthrough</h2> 
<p>This walkthrough takes approximately 30–40 minutes to complete. It assumes familiarity with the AWS CLI, Docker, and basic networking concepts.</p> 
<h2>Prerequisites</h2> 
<p>Before you begin, make sure you have:</p> 
<ul> 
 <li>An AWS account. If you don’t have one, you can create a new <a href="https://portal.aws.amazon.com/billing/signup" rel="noopener noreferrer" target="_blank">AWS account</a>.</li> 
 <li>AWS Command Line Interface (AWS CLI) v2 version 2.32.0 or later. Run <code>aws --version</code> to verify your installation. For installation instructions, see <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener noreferrer" target="_blank">Installing or updating the AWS CLI</a>.</li> 
 <li>Docker version 20.10 or later with buildx support. Run<code>docker --version</code> to verify. For installation instructions, see the <a href="https://docs.docker.com/get-docker/" rel="noopener noreferrer" target="_blank">Docker documentation</a>.</li> 
 <li>Git to clone the sample repository. For installation instructions, see <a href="https://git-scm.com/downloads" rel="noopener noreferrer" target="_blank">Git downloads</a>.</li> 
 <li>jq for parsing JSON output from the AWS CLI. Run <code>jq --version</code> to verify. For installation instructions, see the <a href="https://jqlang.github.io/jq/download/" rel="noopener noreferrer" target="_blank">jq documentation</a>.</li> 
 <li>Bash shell – macOS/Linux terminal or <a href="https://learn.microsoft.com/en-us/windows/wsl/install" rel="noopener noreferrer" target="_blank">Windows Subsystem for Linux (WSL)</a> on Windows</li> 
 <li>Amazon Bedrock model access for the Amazon Nova 2 Lite model in your AWS account. To enable access, follow the instructions in <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener noreferrer" target="_blank">Manage access to Amazon Bedrock foundation models</a>.</li> 
</ul> 
<h3>Step 1: Clone repository and set variables</h3> 
<p>Start by cloning the sample repository and configuring the environment variables used throughout this walkthrough. Open a terminal and run the following commands:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">git clone https://github.com/aws-samples/sample-mcp-server-on-ecs.git</code></pre> 
</div> 
<p>You may need to use <a href="https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens" rel="noopener noreferrer" target="_blank">GitHub Personal Access Token</a> for the preceding step.</p> 
<div class="hide-language"> 
 <pre><code class="lang-javascript">cd sample-mcp-server-on-ecs

# Set your variables (modify these for your environment)
export STACK_NAME=ecs-mcp-blog
export AWS_REGION=us-west-2
export AWS_PROFILE=default</code></pre> 
</div> 
<h3>Step 2: Deploy Infrastructure</h3> 
<p>Deploy all required infrastructure using the provided <a href="https://aws.amazon.com/cloudformation/" rel="noopener noreferrer" target="_blank">AWS CloudFormation</a> template. The template provisions the following resources in a single stack:</p> 
<ul> 
 <li>An <a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud</a> (Amazon VPC) with public and private subnets across two Availability Zones</li> 
 <li>An <a href="https://aws.amazon.com/ecs/" rel="noopener noreferrer" target="_blank">Amazon ECS</a> cluster with <a href="https://aws.amazon.com/fargate/" rel="noopener noreferrer" target="_blank">AWS Fargate</a> capacity providers</li> 
 <li><a href="https://aws.amazon.com/ecr/" rel="noopener noreferrer" target="_blank">Amazon Elastic Container Registry</a> (Amazon ECR) repositories for the three container images</li> 
 <li>An <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon S3</a> bucket for product catalog data</li> 
 <li><a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (AWS IAM) roles with least-privilege permissions</li> 
 <li>Security groups that restrict traffic between services</li> 
 <li>An <a href="https://aws.amazon.com/cloud-map/" rel="noopener noreferrer" target="_blank">AWS Cloud Map</a> namespace for Service Connect service discovery</li> 
</ul> 
<p>Run the following command to deploy the stack:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws cloudformation deploy \
--template-file cloudformation/infrastructure.yaml \
--stack-name $STACK_NAME \
--capabilities CAPABILITY_NAMED_IAM \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<p>Check that the stack deployed successfully:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code"># Should output CREATE_COMPLETE
aws cloudformation describe-stacks \
--stack-name $STACK_NAME \
--region $AWS_REGION \
--profile $AWS_PROFILE \
--query 'Stacks[0].StackStatus' \
--output text
</code></pre> 
</div> 
<p>The output should read <code>CREATE_COMPLETE</code>.</p> 
<h3>Step 3: Retrieve stack outputs</h3> 
<p>Next, run the provided setup script to export all CloudFormation stack outputs as environment variables. These variables are referenced in subsequent steps to deploy and configure the Amazon ECS services.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">source scripts/setup-env.sh</code></pre> 
</div> 
<p>You should see output confirming all variables are set. If any value shows empty or you see an error, verify the stack completed successfully in Step 2.</p> 
<h3>Step 4: Authenticate with Amazon ECR</h3> 
<p>Authenticate your local Docker client with Amazon ECR:</p> 
<div class="hide-language"> 
 <pre><code class="lang-powershell">aws ecr get-login-password --region $AWS_REGION --profile $AWS_PROFILE | \
docker login --username AWS --password-stdin $ECR_REGISTRY</code></pre> 
</div> 
<p>You should see <code>Login Succeeded</code> in the output. If you receive an authorization error, verify that your AWS CLI credentials have <code>ecr:GetAuthorizationToken</code> permission and that the <code>ECR_REGISTRY</code> variable from Step 3 is set correctly.</p> 
<h3>Step 5: Upload the product catalog</h3> 
<p>Upload the sample product catalog to the Amazon S3 bucket provisioned by the CloudFormation stack. The MCP Server reads this file at runtime to respond to product search queries.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws s3 cp sample-data/product-catalog.json s3://$S3_BUCKET/product-catalog.json \
--region $AWS_REGION --profile $AWS_PROFILE</code></pre> 
</div> 
<p>Confirm the upload worked:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code"># Should return product-catalog.json with size ~5 KiB
aws s3 ls s3://$S3_BUCKET/ --region $AWS_REGION --profile $AWS_PROFILE</code></pre> 
</div> 
<h3>Step 6: Build and push Docker images</h3> 
<p>Build and push container images for all three services to Amazon ECR. The <code>--platform linux/amd64</code> flag is required for AWS Fargate compatibility. See <a href="https://docs.aws.amazon.com/batch/latest/userguide/push-array-image.html" rel="noopener noreferrer" target="_blank">Pushing a Docker image to Amazon ECR</a> for reference.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># MCP Server
docker buildx build --platform linux/amd64 \
-t $ECR_REGISTRY/${STACK_NAME}-mcp-server:latest \
./mcp-server --push

# Agent
docker buildx build --platform linux/amd64 \
-t $ECR_REGISTRY/${STACK_NAME}-agent:latest \
./agent --push

# UI
docker buildx build --platform linux/amd64 \
-t $ECR_REGISTRY/${STACK_NAME}-ui:latest \
./ui --push
</code></pre> 
</div> 
<p>Make sure all images pushed correctly:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># Each should return an imageDigest if empty, the push failed
for repo in mcp-server agent ui; do
echo "--- ${STACK_NAME}-${repo} ---"
aws ecr describe-images \
--repository-name ${STACK_NAME}-${repo} \
--region $AWS_REGION \
--profile $AWS_PROFILE \
--query 'imageDetails[0].[imageTags[0],imageSizeInBytes]' \
--output text
done</code></pre> 
</div> 
<h3>Step 7: Configure Amazon ECS Service Connect</h3> 
<p>Amazon ECS Service Connect lets services communicate by name. It uses <a href="https://docs.aws.amazon.com/cloud-map/latest/dg/what-is-cloud-map.html" rel="noopener noreferrer" target="_blank">AWS Cloud Map</a> for discovery and an <a href="https://www.envoyproxy.io/" rel="noopener noreferrer" target="_blank">Envoy</a> sidecar proxy for routing. Each service needs a JSON configuration that defines its discovery name, port mapping, and log destination. The MCP Server and Agent register as discoverable endpoints so other services can reach them by name (like <a href="http://mcp-server:8080/" rel="noopener noreferrer" target="_blank"><code>http://mcp-server:8080</code></a>). The UI acts as a client, it doesn’t register itself, but uses the Envoy proxy to find the Agent.</p> 
<p>For background information on networking approaches, see <a href="https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/networking-connecting-services.html" rel="noopener noreferrer" target="_blank">Networking between Amazon ECS services in a VPC</a>.</p> 
<p>Run the following script to generate the Service Connect configuration files for all three services:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">./scripts/generate-service-connect-configs.sh</code></pre> 
</div> 
<p>You should see output listing the three generated config files in config/ directory.</p> 
<h3>Step 8: Deploy Amazon ECS services</h3> 
<p>Deploy services in dependency order: MCP Server first, then Agent, then UI. This verifies each service can find the ones it depends on at startup. The MCP Server and Agent run as standard Amazon ECS services in private subnets without public access. Amazon ECS Service Connect handles all communication between them. The UI uses Amazon ECS Express Mode, which automatically creates an Application Load Balancer, target group, and auto-scaling policy.</p> 
<p>For a complete CLI walkthrough, see <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_Fargate.html" rel="noopener noreferrer" target="_blank">Creating an Amazon ECS Linux task for Fargate with the AWS CLI</a>.</p> 
<h4>MCP server:</h4> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">aws ecs create-service \
--cluster $CLUSTER_NAME \
--service-name mcp-server-service \
--task-definition ${STACK_NAME}-mcp-server \
--desired-count 2 \
--launch-type FARGATE \
--network-configuration "awsvpcConfiguration={subnets=[$PRIVATE_SUBNETS],securityGroups=[$MCP_SG],assignPublicIp=DISABLED}" \
--service-connect-configuration file://config/${STACK_NAME}-mcp-server-service-connect.json \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<h4>Agent:</h4> 
<div class="hide-language"> 
 <pre><code class="lang-typescript">aws ecs create-service \
--cluster $CLUSTER_NAME \
--service-name agent-service \
--task-definition ${STACK_NAME}-agent \
--desired-count 1 \
--launch-type FARGATE \
--network-configuration "awsvpcConfiguration={subnets=[$PRIVATE_SUBNETS],securityGroups=[$AGENT_SG],assignPublicIp=DISABLED}" \
--service-connect-configuration file://config/${STACK_NAME}-agent-service-connect.json \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<p>Verify both services are running before proceeding:</p> 
<div class="hide-language"> 
 <pre><code class="lang-php">echo "Waiting 60 seconds for tasks to start..."
sleep 60

aws ecs describe-services \
--cluster $CLUSTER_NAME \
--services mcp-server-service agent-service \
--region $AWS_REGION \
--profile $AWS_PROFILE \
--query 'services[].[serviceName,status,runningCount,desiredCount]' \
--output table</code></pre> 
</div> 
<p>Both services should show <code>runningCount:1</code>. If<code>runningCount</code>is 0, check for task failures:</p> 
<div class="hide-language"> 
 <pre><code class="lang-shell">TASK_ARN=$(aws ecs list-tasks --cluster $CLUSTER_NAME --service-name mcp-server-service \
--desired-status STOPPED --region $AWS_REGION --profile $AWS_PROFILE \
--query 'taskArns[0]' --output text)

if [ "$TASK_ARN" != "None" ] &amp;&amp; [ -n "$TASK_ARN" ]; then
aws ecs describe-tasks --cluster $CLUSTER_NAME --tasks $TASK_ARN \
--region $AWS_REGION --profile $AWS_PROFILE \
--query 'tasks[0].[stoppedReason,containers[].reason]' --output text
else
echo "No stopped tasks found — services are running normally."
fi</code></pre> 
</div> 
<h4>UI service (Express Mode)</h4> 
<p>The UI service uses Amazon ECS Express Mode for automated load balancing and public access. Express Mode automatically creates Application Load Balancers, target groups, security groups, and auto-scaling policies. See <a href="https://aws.amazon.com/blogs/aws/build-production-ready-applications-without-infrastructure-complexity-using-amazon-ecs-express-mode/" rel="noopener noreferrer" target="_blank">Build production-ready applications without infrastructure complexity using Amazon ECS Express Mode</a> and the <a href="https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ECSExpressGatewayService.html" rel="noopener noreferrer" target="_blank">API reference</a>.</p> 
<p><strong>Create the UI service:</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-css">aws ecs create-express-gateway-service \
--cluster $CLUSTER_NAME \
--service-name ui-service \
--execution-role-arn $EXECUTION_ROLE \
--infrastructure-role-arn $INFRA_ROLE \
--primary-container "{
\"image\": \"${UI_ECR}:latest\",
\"containerPort\": 7860,
\"awsLogsConfiguration\": {
\"logGroup\": \"${UI_LOG_GROUP}\",
\"logStreamPrefix\": \"ecs\"
},
\"environment\": [
{\"name\": \"AGENT_ENDPOINT\", \"value\": \"http://agent:3000\"}
]
}" \
--task-role-arn $UI_TASK_ROLE \
--network-configuration subnets=$PUBLIC_SUBNETS,securityGroups=$UI_SG \
--cpu "256" \
--memory "512" \
--scaling-target minTaskCount=1,maxTaskCount=4,autoScalingMetric=AVERAGE_CPU,autoScalingTargetValue=70 \
--tags key=Project,value=ECS-MCP-Blog \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<p>Wait for the service to stabilize (around four minutes):</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws ecs wait services-stable \
--cluster $CLUSTER_NAME \
--services ui-service \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<p>Then attach Service Connect to the UI service and force a new deployment:</p> 
<div class="hide-language"> 
 <pre><code class="lang-powershell">aws ecs update-service \
--cluster $CLUSTER_NAME \
--service ui-service \
--service-connect-configuration file://config/${STACK_NAME}-ui-service-connect.json \
--force-new-deployment \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<p><strong>Note: </strong>Allow six to seven minutes after this update for Express Mode’s canary deployment to complete before proceeding.</p> 
<p>Wait for the deployment to complete before proceeding (around six minutes).</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws ecs wait services-stable \
--cluster $CLUSTER_NAME \
--services ui-service \
--region $AWS_REGION \
--profile $AWS_PROFILE</code></pre> 
</div> 
<h3>Step 9: Retrieve the UI endpoint</h3> 
<p>Confirm all services are healthy, then retrieve the public URL for the UI. For the Boto3 SDK reference on <code>describe_express_gateway_service</code>, see the <a href="https://docs.aws.amazon.com/boto3/latest/reference/services/ecs/client/create_express_gateway_service.html" rel="noopener noreferrer" target="_blank">Boto3 ECS documentation.</a></p> 
<div class="hide-language"> 
 <pre><code class="lang-typescript"># Check service status
aws ecs describe-services \
--cluster $CLUSTER_NAME \
--services mcp-server-service agent-service ui-service \
--region $AWS_REGION \
--profile $AWS_PROFILE \
--query 'services[].[serviceName,status,runningCount,desiredCount]' \
--output table

# Get UI public URL via Express Mode API
UI_SERVICE_ARN=$(aws ecs describe-services --cluster $CLUSTER_NAME --services ui-service \
--region $AWS_REGION --profile $AWS_PROFILE \
--query 'services[0].serviceArn' --output text)

UI_ENDPOINT=$(aws ecs describe-express-gateway-service \
--service-arn $UI_SERVICE_ARN \
--region $AWS_REGION --profile $AWS_PROFILE \
--query 'service.activeConfigurations[0].ingressPaths[0].endpoint' --output text)

echo "UI URL: https://${UI_ENDPOINT}/"</code></pre> 
</div> 
<h3>Step 10: Test the application</h3> 
<p>Open the UI URL in your browser and run a few sample queries to validate end-to-end connectivity:</p> 
<ul> 
 <li>“Show me electronics under $100”</li> 
 <li>“What laptops do you have?”</li> 
 <li>“Find running shoes in stock”</li> 
</ul> 
<p>Each query travels the full chain: the UI sends it to the Agent over Amazon ECS Service Connect, the Agent calls Amazon Bedrock to decide which MCP tools to invoke, the MCP Server searches the product catalog in Amazon S3, and the results flow back to the user. Expect a 3 to 5-second response time on the first query as the MCP Server loads the product catalog from S3.If the UI loads but queries return errors, check the Agent and MCP Server logs:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># Agent logs - look for Amazon Bedrock or MCP connection errors
aws logs tail /ecs/${STACK_NAME}/agent --since 10m \
--region $AWS_REGION --profile $AWS_PROFILE

# MCP Server logs - look for Amazon S3 or startup errors
aws logs tail /ecs/${STACK_NAME}/mcp-server --since 10m \
--region $AWS_REGION --profile $AWS_PROFILE</code></pre> 
</div> 
<h2>Operational considerations for running MCP servers on Amazon ECS</h2> 
<p>This section covers operational considerations for running MCP servers on Amazon ECS, focusing on two key areas:</p> 
<p><strong>Security and compliance:</strong></p> 
<p>AI agents need secure access to AWS resources through MCP, with proper access control and permission enforcement for agent-initiated requests. Today, all AWS services authorize requests using AWS-native Signature Version 4 (SigV4), which requires you to sign requests using a symmetric access key and secret. <a href="https://docs.aws.amazon.com/aws-mcp/latest/userguide/security.html" rel="noopener noreferrer" target="_blank">The Amazon ECS MCP server addresses this challenge</a> through a defense-in-depth security architecture that bridges MCP’s OAuth 2.1 authentication standard with AWS’s native IAM/SigV4 access control model.</p> 
<p>This approach provides several security layers:</p> 
<ul> 
 <li>Centralized authentication through AWS Identity and Access Management (AWS IAM)</li> 
 <li>Least-privilege authorization that mirrors calling user permissions</li> 
 <li>Comprehensive audit logs via AWS CloudTrail</li> 
 <li>Sandboxed execution on Amazon ECS with Fargate with minimal task role permissions</li> 
 <li>Input validation for all tool parameters</li> 
</ul> 
<p>To further protect your data, use SSL/TLS (minimum TLS 1.2, TLS 1.3 recommended) for all communication with AWS resources, apply AWS encryption solutions across all services, and never embed confidential or sensitive information, such as customer email addresses, in tags or free-form text fields, as this data may appear in billing or diagnostic logs. See <a href="https://docs.aws.amazon.com/aws-mcp/latest/userguide/data-protection.html" rel="noopener noreferrer" target="_blank">Data protection in AWS MCP Server</a> for additional guidance.</p> 
<p><strong>Monitoring and Observability:</strong></p> 
<p>Maintaining reliability, availability, and performance of your MCP server deployment requires a layered observability strategy. AWS CloudTrail captures all API calls made by or on behalf of your ECS tasks, including Amazon Bedrock <code>InvokeModel</code> requests, Amazon S3 <code>GetObject</code> operations, and AWS Secrets Manager <code>GetSecretValue</code> calls, and delivers log files to an Amazon S3 bucket for audit and forensic analysis. You can identify which users and accounts called AWS, the source IP address, and when each call occurred. Complement this with <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html" rel="noopener noreferrer" target="_blank">Amazon CloudWatch Logs</a> using JSON-structured logging to enable advanced querying with CloudWatch Logs Insights. For example, you can track average tool invocation latency by tool name across the MCP server. Enable <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html" rel="noopener noreferrer" target="_blank">Container Insights</a> for cluster- and task-level metrics including CPU and memory utilization, network throughput, and task availability, and configure CloudWatch alarms to alert proactively when thresholds are breached. For distributed tracing across the three-tier architecture, add the <a href="https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html" rel="noopener noreferrer" target="_blank">AWS X-Ray</a> daemon as a sidecar container and instrument application code using the X-Ray SDK to trace requests end-to-end from the Gradio UI through the Agent to the MCP server. Finally, Amazon ECS Service Connect automatically publishes Envoy proxy metrics, including <code>ActiveConnectionCount</code>, <code>NewConnectionCount</code>, <code>ProcessedBytes</code>, and <code>TargetResponseTime</code> to Amazon CloudWatch, giving you visibility into inter-service communication health and latency distribution across the deployment.</p> 
<h2>Cleanup</h2> 
<p>This deployment creates billable AWS resources. To avoid ongoing charges, run the provided cleanup script, which removes all resources in the correct dependency order: Amazon ECS services, Express Mode resources, Amazon S3 buckets, Amazon ECR repositories, the AWS CloudFormation stack, and retained CloudWatch log groups.</p> 
<div class="hide-language"> 
 <pre><code class="lang-javascript"># Verify your deployment variables are set
export STACK_NAME=ecs-mcp-blog
export AWS_REGION=us-west-2
export AWS_PROFILE=default

# Run cleanup
./scripts/cleanup.sh</code></pre> 
</div> 
<p>The script will:</p> 
<ol> 
 <li>Delete all Amazon ECS services in parallel</li> 
 <li>Wait for services to drain and stop</li> 
 <li>Remove the Express Mode Application Load Balancer and orphaned security groups</li> 
 <li>Empty Amazon S3 buckets including versioned objects</li> 
 <li>Delete Amazon ECR repositories and all container images</li> 
 <li>Delete the AWS CloudFormation stack</li> 
 <li>Remove retained Amazon CloudWatchlog groups</li> 
</ol> 
<p>If any step fails, the script continues and reports failures at the end with links to the AWS Management Console for manual remediation.</p> 
<p>If the CloudFormation stack deletion fails on the VPC resource because of lingering elastic network interfaces or other dependencies, open the <a href="https://console.aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon VPC console</a>, select the VPC tagged with your stack name (<code>ecs-mcp-blog</code>), and choose <strong>Actions &gt; Delete VPC</strong> to remove the VPC and its associated subnets, route tables, internet gateways, and network address translation (NAT) gateways. See <a href="https://docs.aws.amazon.com/vpc/latest/userguide/delete-vpc.html" rel="noopener noreferrer" target="_blank">Delete your VPC</a> for troubleshooting guidance.</p> 
<p>After the script completes, verify that the CloudFormation stack returns <code>DELETE_COMPLETE</code>, the S3 bucket returns a <code>NoSuchBucket</code> error, and the ECR repository query returns an empty list.</p> 
<h2>Conclusion</h2> 
<p>Amazon ECS with AWS Fargate gives you the operational control, networking flexibility, and connection management you need to run MCP servers in production. By combining ECS Service Connect for service-to-service communication, Express Mode for automated load balancing and public endpoints, and a defense-in-depth security architecture that bridges MCP’s OAuth 2.1 standard with AWS’s native IAM/SigV4 model, you can build AI agent infrastructure that is secure, observable, and ready for regulated industries and multi-tenant environments.</p> 
<p>AWS is expanding its catalog of pre-built, production-ready MCP servers, including the <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-mcp-introduction.html" rel="noopener noreferrer" target="_blank">Amazon ECS MCP server</a>, which provides secure, least-privilege access to AWS services through standardized tools, and a growing library of community and AWS-managed servers that accelerate integration across databases, APIs, and enterprise data sources. This ecosystem means teams can move faster by composing agents from pre-built building blocks rather than building every integration from scratch.</p> 
<p>Ready to try it? Clone the <a href="https://github.com/aws-samples/sample-mcp-server-on-ecs" rel="noopener noreferrer" target="_blank">sample repository</a> and adapt the MCP server to your own tools and data sources. The patterns you’ve seen here will work for most AI agent architectures on ECS. For additional guidance, see the <a href="https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/intro.html" rel="noopener noreferrer" target="_blank">Amazon ECS Best Practices Guide</a> and the <a href="https://docs.aws.amazon.com/aws-mcp/latest/userguide/what-is-mcp.html" rel="noopener noreferrer" target="_blank">AWS MCP Server documentation</a>.</p> 
<hr /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft wp-image-20482" height="96" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/07/CONTAINERS-153-2-150x150.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Piyush Mattoo</h3> 
  <p><a href="https://www.linkedin.com/in/piyush-mattoo-aws/">Piyush</a> is a Solutions Architect at Amazon Web Services, where he specializes in enterprise-scale distributed systems and modern software architecture. He works closely with customers to design and implement cloud-native solutions using containers and AWS services.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft wp-image-20483" height="133" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/07/CONTAINERS-153-3-150x150.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Sudheer Manubolu</h3> 
  <p><a href="https://www.linkedin.com/in/sudheer-manubolu/">Sudheer</a> is a Solutions Architect at Amazon Web Services, where he helps organizations modernize and scale their infrastructure on AWS. With a focus on cloud-native and containerized architectures, AI integrations, and developer platforms, he works with customers to turn emerging technologies into practical, production-ready solutions.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft wp-image-20484" height="133" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/04/07/CONTAINERS-153-4-150x150.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Stacey Hou</h3> 
  <p><a href="https://www.linkedin.com/in/staceyhou/">Stacey</a> is a Senior Product Manager – Technical at AWS, where she focuses on generative AI and observability at Amazon Elastic Container Service (ECS). She works closely with customers and engineering teams to drive innovations that improve the experience of building, operating, and troubleshooting containerized applications at AWS.</p> 
 </div> 
</footer>
