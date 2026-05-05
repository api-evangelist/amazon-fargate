---
title: "Automated deployments with GitHub Actions for Amazon ECS Express Mode"
url: "https://aws.amazon.com/blogs/containers/automated-deployments-with-github-actions-for-amazon-ecs-express-mode/"
date: "Tue, 10 Mar 2026 16:32:50 +0000"
author: "Olawale Olaleye"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>Organizations adopt containerized applications for multiple benefits, including accelerated development cycles and faster feature delivery. <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/express-service-overview.html" rel="noopener noreferrer" target="_blank">Amazon Elastic Container Service (Amazon ECS) Express Mode</a> streamlines this by managing infrastructure automatically—networking, load balancing, and capacity provisioning happen without manual configuration. However, building container images, pushing them to registries, and updating services when code changes still require manual coordination or custom scripts.</p> 
<div class="p-client_container"> 
 <div class="p-ia4_client_container"> 
  <div class="p-ia4_client p-ia4_client--sidebar-wide p-ia4_client--with-split-view-feature"> 
   <div class="p-client_workspace_wrapper"> 
    <div class="p-client_workspace"> 
     <div class="p-client_workspace__layout"> 
      <div class="p-client_workspace__tabpanel"> 
       <div class="enabled-managed-focus-container"> 
        <div class="p-view_contents p-view_contents--primary"> 
         <div class="p-flexpane p-flexpane--iap1 p-flexpane--ia_details_popover"> 
          <div class="p-flexpane__body p-threads_flexpane_container p-flexpane__body--light"> 
           <div class="p-file_drag_drop__container p-threads_flexpane"> 
            <div class="p-threads_flexpane__content"> 
             <div class="p-threads_flexpane__message_area"> 
              <div class="flex_one no_min_height"> 
               <div id="C0A70BTRTC4-1772117357.286609-thread-list-Thread"> 
                <div class="c-virtual_list c-virtual_list--scrollbar c-scrollbar" id="C0A70BTRTC4-1772117357.286609-thread-list-Thread"> 
                 <div class="c-scrollbar__hider"> 
                  <div class="c-scrollbar__child"> 
                   <div class="c-virtual_list__scroll_container"> 
                    <div class="c-virtual_list__item" id="C0A70BTRTC4-1772117357.286609-thread-list-Thread_input"> 
                     <div class="p-threads_footer__input_container p-threads_footer__input_container--sticky_composer"> 
                      <div class="p-threads_footer__input p-message_input_unstyled p-message_input_unstyled--attachments-visible"> 
                       <div class="p-message_input__input_container_unstyled c-wysiwyg_container c-wysiwyg_container--theme_light c-wysiwyg_container--with_footer c-wysiwyg_container--theme_light_bordered c-basic_container c-basic_container--size_medium"> 
                        <div class="c-basic_container__body"> 
                         <div class="c-wysiwyg_container__formatting"> 
                          <div class="p-texty_sticky_formatting_bar"> 
                           <div class="p-composer__body p-composer__body--visible p-composer__body--unstyled"> 
                            <div class="c-virtual_list__item" id="C0A70BTRTC4-1772117357.286609-thread-list-Thread_1772124919.807939"> 
                             <div class="c-message_kit__background c-message_kit__background--hovered c-message_kit__background--labels c-message_kit__background--labels--broadcast c-message_kit__message c-message_kit__thread_message"> 
                              <div class="c-message_kit__hover c-message_kit__hover--hovered"> 
                               <div class="c-message_kit__actions c-message_kit__actions--default"> 
                                <div class="c-message_kit__labels c-message_kit__labels--light"> 
                                 <div class="c-message_kit__gutter"> 
                                  <div class="c-message_kit__gutter__right"> 
                                   <div class="c-message_kit__blocks c-message_kit__blocks--rich_text"> 
                                    <div class="c-message__message_blocks c-message__message_blocks--rich_text"> 
                                     <div class="p-block_kit_renderer"> 
                                      <div class="p-block_kit_renderer__block_wrapper p-block_kit_renderer__block_wrapper--first"> 
                                       <div class="p-rich_text_block" dir="auto">
                                        Recently, AWS announced the 
                                        <a href="https://github.com/marketplace/actions/amazon-ecs-deploy-express-service-action-for-github-actions" rel="noopener noreferrer" target="_blank">Amazon ECS “Deploy Express Service” Action for GitHub Actions</a>, a new 
                                        <a href="https://docs.github.com/en/billing/concepts/product-billing/github-actions" rel="noopener noreferrer" target="_blank">GitHub Actions</a> that automates deployments for Amazon ECS Express Mode services. The “Deploy Express Service” GitHub action integrates directly into your GitHub workflows, and when paired with GitHub actions to 
                                        <a href="https://github.com/marketplace/actions/build-and-push-docker-images" rel="noopener noreferrer" target="_blank">build and push container images</a>, can be used to create a seamless continuous deployment pipeline. The “Deploy Express Services” GitHub Action authenticates with 
                                        <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> roles with 
                                        <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html" rel="noopener noreferrer" target="_blank">OpenID Connect (OIDC) authentication</a>, so you don’t store AWS credentials in your repository.
                                       </div> 
                                      </div> 
                                     </div> 
                                    </div> 
                                   </div> 
                                  </div> 
                                 </div> 
                                </div> 
                               </div> 
                              </div> 
                             </div> 
                            </div> 
                           </div> 
                          </div> 
                         </div> 
                        </div> 
                       </div> 
                      </div> 
                     </div> 
                    </div> 
                   </div> 
                  </div> 
                 </div> 
                </div> 
               </div> 
              </div> 
             </div> 
            </div> 
           </div> 
          </div> 
         </div> 
        </div> 
       </div> 
      </div> 
     </div> 
    </div> 
   </div> 
  </div> 
 </div> 
</div> 
<p>In this post, we will walk you through building an automated deployment pipeline using GitHub Actions. You will create a workflow that triggers on code changes, builds Docker images, pushes them to Amazon ECR, and deploys to Amazon ECS Express Mode using IAM roles for secure authentication. By the end, you will have a continuous integration and continuous delivery (CI/CD) workflow that automatically deploys your application when you push code.</p> 
<h2>Architecture overview</h2> 
<p>The following diagram shows how GitHub Actions integrates with Amazon ECS Express Mode to create an automated deployment pipeline:</p> 
<div class="wp-caption alignnone" id="attachment_20197" style="width: 964px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/image-1-5.png"><img alt="" class="wp-image-20197 size-full" height="441" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/image-1-5.png" width="954" /></a>
 <p class="wp-caption-text" id="caption-attachment-20197">Figure 1 – CI/CD architecture diagram showing GitHub Actions workflow</p>
</div> 
<p>When you commit code to your repository on the GitHub website, GitHub Actions automatically builds your container image and pushes it to Amazon ECR. The workflow then updates your Amazon ECS Express Mode service with the new image. An <a href="https://aws.amazon.com/elasticloadbalancing/application-load-balancer/" rel="noopener noreferrer" target="_blank">Application Load Balancer (ALB)</a> routes user traffic to your containerized application running on Amazon ECS.</p> 
<p>This architecture uses OpenID Connect (OIDC) to securely authenticate GitHub Actions with AWS. Because OIDC generates credentials that automatically expire after each workflow run, you don’t need to store long-lived AWS credentials in your repository to reduce your security risk exposure.</p> 
<h2>Prerequisites</h2> 
<p>Before you start, make sure that you have:</p> 
<ul> 
 <li>An AWS account with permissions to create IAM roles, policies, Amazon ECR repositories, and Amazon ECS resources.</li> 
 <li>A <a href="https://github.com/" rel="noopener noreferrer" target="_blank">GitHub account</a>.</li> 
 <li>An <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener noreferrer" target="_blank">AWS Command Line Interface (AWS CLI) installed and configured</a> on your local machine.</li> 
 <li>An Amazon ECR repository to store your container images. If you haven’t created one yet, see <a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html" rel="noopener noreferrer" target="_blank">Creating a private repository</a>.</li> 
 <li>A default <a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud</a> (Amazon VPC) and default subnets, otherwise, see <a href="https://docs.aws.amazon.com/vpc/latest/userguide/work-with-default-vpc.html#create-default-vpc" rel="noopener noreferrer" target="_blank">Create a default VPC</a>. For sensitive workloads, consider creating a custom <a href="https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html" rel="noopener noreferrer" target="_blank">VPC with both public and private subnets</a>.</li> 
 <li>Be comfortable writing <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html" rel="noopener noreferrer" target="_blank">Dockerfiles</a>, understand GitHub Actions workflow syntax, and know how to navigate the AWS console.</li> 
 <li>(Optional) A domain name if you want to use a custom domain with your application.</li> 
</ul> 
<p><strong>Estimated time:</strong>&nbsp;20–30 minutes<br /> <strong>Estimated cost:</strong> Costs vary based on usage. You will incur charges for Amazon ECS tasks, Amazon ECR storage, and data transfer. GitHub Actions usage has no charge for public repositories. Remember to clean up resources after testing.</p> 
<h2>Walkthrough</h2> 
<p>Fork and clone the source code repository on GitHub: <a href="https://github.com/aws-samples/sample-amazon-ecs-express-github-actions.git" rel="noopener noreferrer" target="_blank">sample-amazon-ecs-express-github-actions</a> using the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">git clone&nbsp;https://github.com/YOUR_USERNAME/sample-amazon-ecs-express-github-actions.git
cd&nbsp;sample-amazon-ecs-express-github-actions&nbsp;</code></pre> 
</div> 
<p><strong>Note:</strong> If you forked this repository, GitHub Actions workflows are disabled by default. To enable them, navigate to the <strong>Actions</strong> tab in your forked repository and choose <strong>I understand my workflows, go ahead and enable them</strong>.</p> 
<p>The repository structure looks like this:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">your-app/
├── Dockerfile
└── .github/
&nbsp; &nbsp;&nbsp;└── workflows/
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;└── deploy.yml</code></pre> 
</div> 
<p>The example Dockerfile in this repository pulls the <a href="https://github.com/aws-containers/retail-store-sample-app/tree/main" rel="noopener noreferrer" target="_blank">AWS Containers Retail Sample</a>&nbsp;UI application’s base image from the <a href="https://gallery.ecr.aws/aws-containers/retail-store-sample-ui" rel="noopener noreferrer" target="_blank">Amazon ECR Public Gallery</a>. The application runs on container port 8080 and provides a functional web interface that you can use to verify that your deployment pipeline works correctly.</p> 
<h2>Set up IAM authentication for GitHub Actions</h2> 
<p>GitHub Actions requires permission to interact with your AWS account. You can use OpenID Connect (OIDC) to establish a trust relationship between GitHub and AWS. This approach improves your security posture because it generates short-lived credentials automatically.</p> 
<ol> 
 <li>Create an OIDC provider</li> 
</ol> 
<p>Begin by registering GitHub’s OIDC identity provider with your AWS account with the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws&nbsp;iam&nbsp;create-open-id-connect-provider --url https://token.actions.githubusercontent.com --client-id-list&nbsp;sts.amazonaws.com
</code></pre> 
</div> 
<p>This command registers GitHub’s OIDC provider with your AWS account. AWS will now accept tokens from GitHub Actions and allow role assumption.</p> 
<ol start="2"> 
 <li>Create the IAM trust policy</li> 
</ol> 
<p>Create a trust policy that specifies exactly which GitHub repositories can assume your IAM role. Replace <code>YOUR_ACCOUNT_ID</code>, <code>YOUR_GITHUB_USERNAME</code>, and <code>YOUR_REPO_NAME</code> with your actual values:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
&nbsp;&nbsp;"Version": "2012-10-17",
&nbsp;&nbsp;"Statement": [
&nbsp;&nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp;"Principal": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
&nbsp;&nbsp; &nbsp; &nbsp;},
&nbsp;&nbsp; &nbsp; &nbsp;"Action": "sts:AssumeRoleWithWebIdentity",
&nbsp;&nbsp; &nbsp; &nbsp;"Condition": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"StringEquals": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;},
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"StringLike": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/YOUR_REPO_NAME:*"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;}
&nbsp;&nbsp;]
}
</code></pre> 
</div> 
<p>Save this as <code>trust-policy.json</code>. The <code>StringLike</code> condition restricts the role assumption to your specific repository, preventing other GitHub repositories from using this role.</p> 
<ol start="3"> 
 <li>Create the IAM role</li> 
</ol> 
<p>Create an IAM role with the trust policy using the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws&nbsp;iam&nbsp;create-role&nbsp;\
&nbsp;&nbsp; &nbsp;--role-name github-actions-ecs-role \
&nbsp;&nbsp; &nbsp;--assume-role-policy-document&nbsp;file://trust-policy.json
</code></pre> 
</div> 
<p>GitHub Actions will assume this role during your workflow execution.</p> 
<ol start="4"> 
 <li>Create permission policies</li> 
</ol> 
<p>GitHub Actions needs specific permissions to build and deploy your application. Following the principle of least privilege, define two separate policies that grant only the minimum required permissions.</p> 
<p>The first policy grants permissions for Amazon ECS Express Mode operations. Replace <code>YOUR_ACCOUNT_ID</code>&nbsp;with your actual value and save the following as <code>ecs-express-policy.json</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
&nbsp;&nbsp; &nbsp;"Version": "2012-10-17",
&nbsp;&nbsp; &nbsp;"Statement": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:CreateCluster",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:RegisterTaskDefinition",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:CreateExpressGatewayService",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:UpdateExpressGatewayService",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:DescribeExpressGatewayService",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:DescribeClusters",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:DescribeServices",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:ListServiceDeployments",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:UpdateService",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecs:DescribeServiceDeployments"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Resource": "*"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;},
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Action": "iam:PassRole",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Resource": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsTaskExecutionRole",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsInfrastructureRoleForExpressServices"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Condition": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"StringEquals": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"iam:PassedToService": "ecs.amazonaws.com"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;]
}
</code></pre> 
</div> 
<p>The second policy grants permissions for Amazon ECR operations. Replace <strong>region</strong>, <strong>accound-id</strong>, and <strong>my-repo</strong> with your AWS Region, Account ID, and repository name. Save the following <code>ecr-policy.json</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
&nbsp;&nbsp; &nbsp;"Version": "2012-10-17",
&nbsp;&nbsp; &nbsp;"Statement": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Sid": "AllowPushPull",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:BatchGetImage",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:BatchCheckLayerAvailability",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:CompleteLayerUpload",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:GetDownloadUrlForLayer",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:InitiateLayerUpload",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:PutImage",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:UploadLayerPart"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Resource":"arn:aws:ecr:::repository/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;},
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Sid": "AllowLogin",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"ecr:GetAuthorizationToken"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Resource": "*"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;]
}
</code></pre> 
</div> 
<p>These policies grant the minimum permissions required to push images to Amazon ECR and deploy to Amazon ECS Express Mode.</p> 
<ol start="5"> 
 <li>Attach policies to the role</li> 
</ol> 
<p>Attach both policies to your IAM role using the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># Create and attach ECS Express policy
aws&nbsp;iam&nbsp;create-policy&nbsp;\
&nbsp;&nbsp; &nbsp;--policy-name GitHubActionsECSExpressPolicy \
&nbsp;&nbsp; &nbsp;--policy-document&nbsp;file://ecs-express-policy.json

aws&nbsp;iam&nbsp;attach-role-policy&nbsp;\
&nbsp;&nbsp; &nbsp;--role-name github-actions-ecs-role \
&nbsp;&nbsp; &nbsp;--policy-arn&nbsp;arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECSExpressPolicy

# Create and attach ECR policy
aws&nbsp;iam&nbsp;create-policy&nbsp;\
&nbsp;&nbsp; &nbsp;--policy-name GitHubActionsECRPolicy \
&nbsp;&nbsp; &nbsp;--policy-document&nbsp;file://ecr-policy.json

aws&nbsp;iam&nbsp;attach-role-policy&nbsp;\
&nbsp;&nbsp; &nbsp;--role-name github-actions-ecs-role \
&nbsp;&nbsp; &nbsp;--policy-arn&nbsp;arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECRPolicy
</code></pre> 
</div> 
<h2>Create IAM roles for Amazon ECS Express Mode</h2> 
<p>Amazon ECS Express Mode requires two additional IAM roles to function: an <strong>execution role</strong> that allows Amazon ECS to pull container images and write logs (<em>ecsTaskExecutionRole</em>), and an <strong>infrastructure role</strong> that manages the underlying AWS resources (<em>ecsInfrastructureRoleForExpressServices</em>). If you haven’t created these roles yet, follow the instructions in the <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/express-service-getting-started.html#express-service-create-execution-role" rel="noopener noreferrer" target="_blank">Amazon ECS documentation for creating IAM roles</a>.</p> 
<h2>Configure GitHub repository variables</h2> 
<p>Your GitHub Actions workflow references your AWS account details and resource names through repository variables. These values aren’t sensitive, so you can store them as variables rather than secrets, making them easier to reference in your workflow file.Navigate to your GitHub repository on the GitHub website.</p> 
<ol> 
 <li>Go to <strong>Settings</strong>, choose <strong>Secrets and variables</strong>, choose <strong>Actions</strong>, choose <strong>Variables</strong> tab.</li> 
 <li>Add each of the following variables by selecting <strong>New repository variable</strong>:</li> 
</ol> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px;"><strong><br /> Variable name</strong></td> 
   <td style="padding: 10px;"><strong><br /> Example value</strong></td> 
   <td style="padding: 10px;"><strong><br /> Description</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;"><code>AWS_REGION</code></td> 
   <td style="padding: 10px;"><code>us-east-1</code></td> 
   <td style="padding: 10px;">AWS Region where your resources are deployed</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;"><code>AWS_ACCOUNT_ID</code></td> 
   <td style="padding: 10px;">123456789012</td> 
   <td style="padding: 10px;">Your 12-digit AWS account ID</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;"><code>ECR_REPOSITORY</code></td> 
   <td style="padding: 10px;"><code>my-app</code></td> 
   <td style="padding: 10px;">Name of your Amazon ECR repository</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;"><code>ECS_SERVICE</code></td> 
   <td style="padding: 10px;"><code>my-app-service</code></td> 
   <td style="padding: 10px;">Name for your Amazon ECS service</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;"><code>ECS_CLUSTER</code></td> 
   <td style="padding: 10px;"><code>default</code></td> 
   <td style="padding: 10px;">Name for your Amazon ECS cluster</td> 
  </tr> 
 </tbody> 
</table> 
<p><strong>Note</strong>: To specify an existing Amazon ECS cluster name other than <strong>default</strong>, you must <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-cluster-console-v2.html" rel="noopener noreferrer" target="_blank">create the cluster</a> beforehand.</p> 
<p>With these variables configured, you can update your deployment configuration without modifying the workflow code itself.</p> 
<h2>The GitHub Actions workflow</h2> 
<p>The workflow file defines your automated deployment pipeline, specifying what happens when you push code to your repository. This workflow triggers on every push to your main branch and on pull requests, ensuring your changes are automatically built and deployed.</p> 
<p>In your repository, locate the <code>.github/workflows</code> directory to review the example <code>deploy.yml</code> file. The following explains how the key components work together in this workflow:</p> 
<p>The <strong>permissions block</strong> grants id-token write access, and the workflow must request OIDC tokens for AWS authentication. Without this permission, the authentication step would fail.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">jobs:
&nbsp;&nbsp;deploy:
&nbsp;&nbsp; &nbsp;name: Deploy
&nbsp;&nbsp; &nbsp;runs-on: ubuntu-latest
&nbsp;&nbsp; &nbsp;environment: production
&nbsp;&nbsp; &nbsp;
&nbsp;&nbsp; &nbsp;# OIDC authentication requires write permission for id-token
&nbsp;&nbsp; &nbsp;permissions:
&nbsp;&nbsp; &nbsp; &nbsp;id-token: write
&nbsp;&nbsp; &nbsp; &nbsp;contents: read</code></pre> 
</div> 
<p>The <strong>Configure AWS credentials step</strong> uses OIDC to authenticate with AWS by assuming the IAM role that you created earlier. AWS generates temporary credentials that automatically expire after the workflow completes, typically within an hour.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">&nbsp; &nbsp;&nbsp;# Authenticate with AWS using OIDC
&nbsp;&nbsp; &nbsp;- name: Configure AWS credentials
&nbsp;&nbsp; &nbsp; &nbsp;uses: aws-actions/configure-aws-credentials@v5
&nbsp;&nbsp; &nbsp; &nbsp;with:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;aws-region: ${{ env.AWS_REGION }}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;role-to-assume: arn:aws:iam::${{ env.AWS_ACCOUNT_ID }}:role/github-actions-ecs-role
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;role-session-name: GitHubActionsECSDeployment
</code></pre> 
</div> 
<p>The <strong>Build, tag, and push step</strong> tags the container image using the first 7 characters of the commit hash. This SHA-based tagging provides precise version tracking and traceability, following AWS best practices for container image management. Each deployment references a specific, immutable image version, which helps with troubleshooting and enables reliable rollbacks if needed.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">&nbsp;&nbsp; &nbsp;- name: Build, tag, and push image to Amazon ECR
&nbsp;&nbsp; &nbsp; &nbsp;id: build-image
&nbsp;&nbsp; &nbsp; &nbsp;env:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
&nbsp;&nbsp; &nbsp; &nbsp;# nosemgrep: third-party-action-not-pinned-to-commit-sha
&nbsp;&nbsp; &nbsp; &nbsp;uses: docker/build-push-action@v6
&nbsp;&nbsp; &nbsp; &nbsp;with:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;context: .
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;push: true
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;# Tag with commit SHA for version tracking
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;tags: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}</code></pre> 
</div> 
<p>The <strong>Deploy step</strong> uses the <a href="https://github.com/marketplace/actions/amazon-ecs-deploy-express-service-action-for-github-actions" rel="noopener noreferrer" target="_blank">official AWS action</a> to deploy your Amazon ECS Express Mode service with the new container image. This step handles the complexity of task definition updates and service deployments. The action will check if the specified cluster exists (or creates it if default is specified and doesn’t exist), creates the service (or updates it, if the service already exists), and waits for the service and deployment to reach a stable state. This step provisions your complete application stack including an Amazon ECS service with tasks launched on AWS Fargate, an Application Load Balancer with target groups and health checks, auto scaling policies based on CPU utilization, security groups and networking configuration, and a custom domain with an AWS provided URL (it looks like <code>xxxxx.ecs.region.on.aws</code>).</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">&nbsp;&nbsp; &nbsp;- name: Deploy to ECS Express Mode
&nbsp;&nbsp; &nbsp; &nbsp;uses: aws-actions/amazon-ecs-deploy-express-service@v1
&nbsp;&nbsp; &nbsp; &nbsp;env:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
&nbsp;&nbsp; &nbsp; &nbsp;with:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;# Required inputs
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;service-name: ${{ env.ECS_SERVICE }}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;image: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;execution-role-arn: arn:aws:iam::${{ env.AWS_ACCOUNT_ID }}:role/ecsTaskExecutionRole
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;infrastructure-role-arn: arn:aws:iam::${{ env.AWS_ACCOUNT_ID }}:role/ecsInfrastructureRoleForExpressServices
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;# Container configuration
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;container-port: 8080

&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;# Resource configuration
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;task-role-arn: arn:aws:iam::${{ env.AWS_ACCOUNT_ID }}:role/ecsTaskExecutionRole
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;cluster: ${{ env.ECS_CLUSTER }}</code></pre> 
</div> 
<p><strong>Tip</strong>: This deployment step uses minimal configuration. You can customize your deployment with additional options like environment variables, health check settings, scaling policies, and custom VPC configuration. See the <a href="https://github.com/marketplace/actions/amazon-ecs-deploy-express-service-action-for-github-actions#complete-example-with-all-options" rel="noopener noreferrer" target="_blank">Complete Example with All Options</a> for all available options.</p> 
<h2>Test your deployment pipeline</h2> 
<p>Verify that your automated deployment pipeline works correctly by making a small change to your application code. Commit your changes and push to your main branch:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">git add .
git commit -m "Test automated deployment"
git push origin main</code></pre> 
</div> 
<p>Monitor the workflow execution by navigating to the <strong>Actions</strong> tab in your GitHub repository. You will see your workflow running with real-time logs for each step.</p> 
<p>After the workflow completes successfully, verify the deployment in the <a href="https://console.aws.amazon.com/ecs/" rel="noopener noreferrer" target="_blank">Amazon ECS console</a>. Your service will show a new deployment with the “Deployment has completed” as shown in the following image.</p> 
<p><img alt="" class="alignnone size-full wp-image-20203" height="292" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/Containers-154-1.jpg" width="844" /></p> 
<p><em>GitHub Actions workflow execution showing deployment progress</em></p> 
<p>Access your application using the default URL provided in the Amazon ECS console. You can see your updated application running with the changes that you just deployed.</p> 
<p><img alt="" class="alignnone size-full wp-image-20204" height="798" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/containers-154-2.jpg" width="844" /></p> 
<p><em>Amazon ECS console showing successful deployment</em></p> 
<h2>Security best practices</h2> 
<p>This implementation follows AWS security best practices to protect your deployment pipeline and running applications.</p> 
<p><strong>OIDC authentication</strong> generates temporary credentials that automatically expire, reducing the risk of credential exposure.</p> 
<p><strong>Least privilege IAM policies</strong> grant only the permissions needed for the deployment workflow. We recommend reviewing and tightening these policies based on your specific requirements. For example, you might restrict the Resource field to specific Amazon ECR repositories or Amazon ECS clusters rather than using wildcards.</p> 
<p><strong>You can use GitHub environments</strong> to add protection rules like required reviewers for production deployments. The workflow uses the production environment, which you can configure to require manual approval before deploying to production.</p> 
<h2>Cleaning up</h2> 
<p>To avoid incurring future charges, delete the resources that you created during this tutorial.</p> 
<p><strong>Delete the Amazon ECS Express Mode service:</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws ecs delete-express-gateway-service --service-arn &lt;service-arn&gt;</code></pre> 
</div> 
<p><strong>(Optional) Delete the Amazon ECS cluster:</strong></p> 
<p><code>aws ecs delete-cluster --cluster $ECS_CLUSTER</code></p> 
<p><strong>Note</strong>: Only delete the cluster if the GitHub Actions workflow created it for you. If you’re using a pre-existing cluster with other services, skip this step to avoid disrupting those services.</p> 
<p><strong>Delete container images from Amazon ECR:</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws ecr batch-delete-image \
&nbsp;&nbsp;&nbsp; --repository-name $ECR_REPOSITORY \
&nbsp;&nbsp;&nbsp; --image-ids "$(aws ecr list-images --repository-name $ECR_REPOSITORY --query 'imageIds[*]' --output json)"


aws ecr delete-repository --repository-name $ECR_REPOSITORY</code></pre> 
</div> 
<p><strong>Remove IAM resources:</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-css">aws iam detach-role-policy \
&nbsp;&nbsp;&nbsp; --role-name github-actions-ecs-role \
&nbsp;&nbsp;&nbsp; --policy-arn arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECSExpressPolicy

aws iam detach-role-policy \
&nbsp;&nbsp;&nbsp; --role-name github-actions-ecs-role \
&nbsp;&nbsp;&nbsp; --policy-arn arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECRPolicy


aws iam delete-policy \
&nbsp;&nbsp;&nbsp; --policy-arn arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECSExpressPolicy

aws iam delete-policy \
&nbsp;&nbsp;&nbsp; --policy-arn arn:aws:iam::${YOUR_ACCOUNT_ID}:policy/GitHubActionsECRPolicy


aws iam delete-role --role-name github-actions-ecs-role


aws iam delete-open-id-connect-provider \
&nbsp;&nbsp;&nbsp; --open-id-connect-provider-arn arn:aws:iam::${YOUR_ACCOUNT_ID}:oidc-provider/token.actions.githubusercontent.com</code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>In this post, we walked you through implementing automated deployments for Amazon ECS Express Mode using GitHub Actions. You built a CI/CD pipeline that automatically builds and deploys your containerized applications.The pipeline uses OIDC for secure authentication with temporary credentials to automatically trigger deployments when you commit code, to track versions through commit-based image tagging, and for rollback capabilities using Amazon ECS deployment history. You can extend this foundation by adding automated tests before deployment, implementing approval workflows for production releases, or integrating security scanning tools to catch vulnerabilities before they reach production.</p> 
<p>To learn more, see <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/express-service-overview.html" rel="noopener noreferrer" target="_blank">Amazon ECS Express Mode documentation</a>, <a href="https://github.com/aws-actions" rel="noopener noreferrer" target="_blank">GitHub Actions for AWS</a>, <a href="https://github.com/marketplace/actions/amazon-ecs-deploy-express-service-action-for-github-actions" rel="noopener noreferrer" target="_blank">Amazon ECS Deploy Express Service GitHub Action</a>, and <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/express-service-best-practices.html" rel="noopener noreferrer" target="_blank">Best practices for Amazon ECS Express Mode services</a>.</p> 
<table border="2px" cellpadding="10px" style="border-color: #FF9900;"> 
 <tbody> 
  <tr> 
   <td>Interested in hands-on experience about ECS? <a href="https://aws-experience.com/amer/smb/events/series/Get-Hands-On-With-ECS?trk=576538d8-87d8-4c6b-b9a7-90869f975851&amp;sc_channel=el" rel="noopener noreferrer" target="_blank">Register for a guided hands-on workshop</a></td> 
  </tr> 
 </tbody> 
</table> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="alignleft wp-image-20208 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/image-4-2-150x150.png" width="150" />Olawale Olaleye</strong> is a Sr. GenAI/ML Specialist Solutions Architect at AWS, based in Ireland, and a certified Kubestronaut. With extensive experience architecting enterprise-scale containerized workloads, he specializes in helping organizations modernize their infrastructure across containers, AI/ML operations, and cloud security. Connect with him on <a href="https://www.linkedin.com/in/olawale-olaleye/" rel="noopener noreferrer" target="_blank">LinkedIn.</a></p> 
<p style="clear: both;"><strong><img alt="" class="alignleft size-thumbnail wp-image-20207" height="150" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/02/25/image-5-150x150.jpeg" width="150" />Guillaume Delacour</strong> is a Sr. Specialist Solutions Architect at AWS, based in France,<br /> where he works with digital native customers for 3 years. His interests include application modernization,&nbsp;AWS Container technologies, and developer experience. You can connect with him on&nbsp;<a href="https://www.linkedin.com/in/guillaume-delacour-331b61187/" rel="noopener noreferrer" target="_blank">LinkedIn.</a></p>
