---
title: "Deploy production generative AI at the edge using Amazon EKS Hybrid Nodes with NVIDIA DGX"
url: "https://aws.amazon.com/blogs/containers/deploy-production-generative-ai-at-the-edge-using-amazon-eks-hybrid-nodes-with-nvidia-dgx/"
date: "Wed, 18 Mar 2026 18:57:10 +0000"
author: "Sheng Chen"
feed_url: "https://aws.amazon.com/blogs/containers/feed/"
---
<p>Modern generative AI applications require deployment closer to where data is generated and business decisions are made, but this creates new infrastructure challenges. Organizations in manufacturing, healthcare, finance, and telecommunications need to deliver low-latency, energy-efficient AI workloads at the edge while maintaining data locality and regulatory compliance. However,&nbsp;managing Kubernetes on-premises adds operational complexity that can slow down innovation.</p> 
<p>You can use Amazon Elastic Kubernetes Service (<a href="https://aws.amazon.com/eks/" rel="noopener noreferrer" target="_blank">Amazon EKS</a>)&nbsp;<a href="https://aws.amazon.com/eks/hybrid-nodes/" rel="noopener noreferrer" target="_blank">Hybrid Nodes</a> to address this by joining on-premises infrastructure to the Amazon EKS control plane as remote nodes.&nbsp;This allows you to accelerate AI workload deployment with consistent operational practices, while addressing latency, compliance, and data residency requirements. EKS Hybrid Nodes removes the complexity and burden of self-managing Kubernetes on-premises so that your team can focus on deploying AI applications and driving innovations. It provides unified workflows and tooling alongside centralized monitoring and enhanced observability across your distributed infrastructure.</p> 
<p>EKS Hybrid Nodes enables you to deliver AI capabilities wherever your business demands, such as the following use cases:</p> 
<ul> 
 <li>Run low-latency services at on-premises locations, including real-time inference at the edge</li> 
 <li>Train models with data that must remain on-premises to meet regulatory compliance requirements</li> 
 <li>Deploy inference workloads near source data, such as Retrieval-Augmented Generation (RAG) applications using a local knowledge base</li> 
 <li>Repurpose existing hardware investment</li> 
</ul> 
<p>This post demonstrates a real-world example of integrating EKS Hybrid Nodes with <a href="https://www.nvidia.com/en-au/products/workstations/dgx-spark/" rel="noopener noreferrer" target="_blank">NVIDIA DGX Spark</a>, a compact and energy-efficient GPU platform optimized for edge AI deployment. In this post we walk you through deploying a large language model (LLM) for low-latency generative AI inference on-premises, setting up node monitoring and GPU observability with centralized management through Amazon EKS. Although this post uses DGX Spark, the architecture and patterns discussed apply to other NVIDIA DGX systems or GPU platforms.</p> 
<h2>Solution overview</h2> 
<p>For this demo walkthrough, you create an EKS cluster with EKS Hybrid Nodes enabled, and connect an on-premises DGX Spark as a hybrid node.&nbsp;You install the&nbsp;<a href="https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/overview.html" rel="noopener noreferrer" target="_blank">NVIDIA GPU Operator</a> for Kubernetes to provision GPU resources for the local generative AI inference. Then, you deploy an LLM on the hybrid nodes using&nbsp;<a href="https://docs.nvidia.com/nim/index.html" rel="noopener noreferrer" target="_blank">NVIDIA NIM</a>, which&nbsp;are a set of microservices optimized by NVIDIA for accelerated model deployment. You also set up the Amazon&nbsp;EKS&nbsp;<a href="https://docs.aws.amazon.com/eks/latest/userguide/node-health.html#node-monitoring-agent" rel="noopener noreferrer" target="_blank">Node Monitoring Agent&nbsp;(NMA)</a>&nbsp;to monitor node health and&nbsp;detect GPU-specific issues.&nbsp;Finally, you integrate the&nbsp;NVIDIA <a href="https://docs.nvidia.com/datacenter/dcgm/latest/gpu-telemetry/dcgm-exporter.html" rel="noopener noreferrer" target="_blank">Data Center GPU Manager (DCGM) Exporter</a>&nbsp;with <a href="https://aws.amazon.com/prometheus/" rel="noopener noreferrer" target="_blank">Amazon Managed Service for Prometheus</a> and <a href="https://aws.amazon.com/grafana/" rel="noopener noreferrer" target="_blank">Amazon Managed Grafana </a>to provide&nbsp;GPU metrics observability across hybrid nodes.</p> 
<p>The following diagram presents a high-level overview of the architecture of our solution.</p> 
<div class="wp-caption alignnone" id="wp-image-20275" style="width: 1440px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-image-1.png"><img alt="Figure 1: Hybrid architecture for deploying GenAI workloads on-premises or at the edge using Amazon EKS Hybrid Nodes with NVIDIA DGX" class="size-full wp-image-20275" height="571" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-image-1.png" width="1430" /></a>
 <p class="wp-caption-text" id="caption-wp-image-20275">Figure 1: Hybrid architecture for deploying GenAI workloads on-premises or at the edge using Amazon EKS Hybrid Nodes with NVIDIA DGX</p>
</div> 
<p>EKS Hybrid Nodes requires private network connectivity between your on-premises or edge environment and the <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/" rel="noopener noreferrer" target="_blank">AWS Region</a>. This connectivity can be established using either&nbsp;<a href="https://aws.amazon.com/directconnect/" rel="noopener noreferrer" target="_blank">AWS Direct Connect</a>&nbsp;or&nbsp;<a href="https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html" rel="noopener noreferrer" target="_blank">AWS Site-to-Site VPN</a> into your&nbsp;<a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud (Amazon VPC)</a>. The node and pod&nbsp;Classless Inter-Domain Routing (CIDR) blocks for your hybrid nodes and container workloads must be unique and routable&nbsp;across your network environment. You provide these CIDRs as the&nbsp;<code>RemoteNodeNetwork</code>&nbsp;and&nbsp;<code>RemotePodNetwork</code>&nbsp;values when creating the EKS cluster with hybrid nodes.</p> 
<p>This walkthrough doesn’t cover hybrid networking prerequisites for EKS Hybrid Nodes. Go to the <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-prereqs.html" rel="noopener noreferrer" target="_blank">Amazon EKS user guide</a> for the details.</p> 
<h2>Prerequisites</h2> 
<p>The following prerequisites are necessary to complete this solution:</p> 
<ul> 
 <li>Amazon VPC with two private and two public subnets, across two <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones" rel="noopener noreferrer" target="_blank">Availability Zones (AZs)</a>.</li> 
 <li>An EKS cluster with hybrid nodes enabled.&nbsp;Follow the Amazon <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cluster-create.html" rel="noopener noreferrer" target="_blank">EKS user guide</a>&nbsp;to deploy.</li> 
 <li>On-premises compute nodes running a <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-os.html" rel="noopener noreferrer" target="_blank">compatible operating system</a>.</li> 
 <li>Private connectivity between the on-premises network and Amazon VPC (through VPN or Direct Connect).</li> 
 <li>Two routable RFC-1918 or CGNAT CIDR blocks for <code>RemoteNodeNetwork</code> and <code>RemotePodNetwork</code>.</li> 
 <li>Configure the on-premises firewall and the EKS cluster security groups to allow bi-directional communications between the Amazon EKS control plane and remote node and pod CIDRs, as per the <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-networking.html" rel="noopener noreferrer" target="_blank">networking prerequisites</a>.</li> 
 <li>NVIDIA DGX (or other GPU-enabled) systems as hybrid nodes.</li> 
 <li>NVIDIA NGC account and API key for accessing NIMs, see the NVIDIA <a href="https://docs.nvidia.com/ngc/latest/ngc-user-guide.html" rel="noopener noreferrer" target="_blank">documentation</a>.</li> 
 <li>The following tools: 
  <ul> 
   <li><a href="https://helm.sh/docs/intro/install/" rel="noopener noreferrer" target="_blank">Helm 3.9+</a></li> 
   <li><a href="https://kubernetes.io/docs/tasks/tools/" rel="noopener noreferrer" target="_blank">kubectl</a></li> 
   <li><a href="https://aws.amazon.com/cli/" rel="noopener noreferrer" target="_blank">AWS Command Line Interface (AWS CLI)</a></li> 
   <li><a href="https://eksctl.io/installation/" rel="noopener noreferrer" target="_blank">eksctl CLI</a></li> 
  </ul> </li> 
</ul> 
<h2>Walkthrough</h2> 
<p>The following steps walk you through this solution.</p> 
<h3>Prepare EKS Hybrid Nodes</h3> 
<p>The following three sections walk you through preparations for EKS Hybrid Nodes.</p> 
<h4>Prepare IAM credentials</h4> 
<ol> 
 <li>Amazon EKS Hybrid Nodes use temporary <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> credentials provisioned by <a href="https://aws.amazon.com/systems-manager/" rel="noopener noreferrer" target="_blank">AWS Systems Manager</a> hybrid activations or IAM Roles Anywhere to authenticate with the EKS cluster.&nbsp;Follow <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-creds.html" rel="noopener noreferrer" target="_blank">the Amazon EKS user guide</a> to create the required Hybrid Nodes IAM role (<code>AmazonEKSHybridNodesRole</code>) using either one of the two options.</li> 
 <li>Create an Amazon EKS access entry with the Hybrid Nodes IAM role to enable your on-premises nodes to join the cluster. Go to <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cluster-prep.html" rel="noopener noreferrer" target="_blank">Prepare cluster access for hybrid nodes</a> in the Amazon EKS user guide for more details.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws eks create-access-entry \
--cluster-name &lt;CLUSTER_NAME&gt; \
--principal-arn &lt;HYBRID_NODES_ROLE_ARN&gt; \
--type HYBRID_LINUX</code></pre> 
</div> 
<h4>Install nodeadm and join the DGX Spark as hybrid node</h4> 
<ol> 
 <li>Use&nbsp;EKS Hybrid Nodes CLI&nbsp;(<a href="https://github.com/aws/eks-hybrid" rel="noopener noreferrer" target="_blank">nodeadm</a>)&nbsp;to bootstrap and install all required components for&nbsp;your hybrid nodes to join the EKS cluster. This demo uses the ARM64 version of the nodeadm for the DGX Spark.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-code">curl -OL 'https://hybrid-assets.eks.amazonaws.com/releases/latest/bin/linux/arm64/nodeadm'
chmod +x nodeadm
nodeadm install 1.34 --credential-provider ssm</code></pre> 
</div> 
<ol start="2"> 
 <li>Prepare a <code>nodeConfig.yaml</code> configuration file using the temporary IAM credentials generated in the previous section. The following is an example for using Systems Manager hybrid activations for hybrid nodes credentials.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-yaml">apiVersion: node.eks.aws/v1alpha1
kind: NodeConfig
spec:
&nbsp;&nbsp;cluster:
&nbsp;&nbsp; &nbsp;name:&nbsp;&lt;CLUSTER_NAME&gt;
&nbsp;&nbsp; &nbsp;region:&nbsp;&lt;CLUSTER_REGION&gt;
&nbsp;&nbsp;hybrid:
&nbsp;&nbsp; &nbsp;ssm:
&nbsp;&nbsp; &nbsp; &nbsp;activationCode: &lt;SSM_ACTIVATION_CODE&gt;
&nbsp;&nbsp; &nbsp; &nbsp;activationId: &lt;SSM_ACTIVATION_ID&gt;</code></pre> 
</div> 
<ol start="3"> 
 <li>Run the <code>nodeadm init</code> command with your <code>nodeConfig.yaml</code>&nbsp;to join your hybrid nodes to the EKS cluster.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">nodeadm init --config-source file://nodeConfig.yaml</code></pre> 
 <ol start="4"> 
  <li>For mixed GPU and non-GPU hybrid nodes, we recommend that you add a <code>--register-with-taints=nvidia.com/gpu=Exists:NoSchedule</code> <a href="https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/" rel="noopener noreferrer" target="_blank">taint</a> to GPU nodes to maximize GPU resource usage. Refer to the&nbsp;<a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-nodeadm.html" rel="noopener noreferrer" target="_blank">documentation</a>&nbsp;regarding how to modify the kubelet configuration using <code>nodeadm</code>.</li> 
 </ol> 
 <h4>Install Cilium Container Network Interface (CNI)</h4> 
 <ol> 
  <li>Before running workloads on hybrid nodes,&nbsp;you must <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cni.html" rel="noopener noreferrer" target="_blank">install a compatible CNI</a>. For this example, we use Cilium because it’s the<a href="https://aws.amazon.com/about-aws/whats-new/2025/08/expanded-support-cilium-amazon-eks-hybrid-nodes/" rel="noopener noreferrer" target="_blank"> AWS-supported CNI for EKS Hybrid Nodes</a>.</li> 
 </ol> 
 <p>Create a Cilium configuration file:&nbsp;<code>cilium-values.yaml</code>.</p> 
 <div class="hide-language"> 
  <pre><code class="lang-yaml"># BGP Control Plane for LoadBalancer services
bgpControlPlane:
&nbsp;&nbsp;enabled: true

# NodePort services
nodePort:
&nbsp;&nbsp;enabled: true
&nbsp;&nbsp;
# Node affinity - Run Cilium only on hybrid nodes
affinity:
&nbsp;&nbsp;nodeAffinity:
&nbsp;&nbsp; &nbsp;requiredDuringSchedulingIgnoredDuringExecution:
&nbsp;&nbsp; &nbsp; &nbsp;nodeSelectorTerms:
&nbsp;&nbsp; &nbsp; &nbsp;- matchExpressions:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;- key: eks.amazonaws.com/compute-type
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;operator: In
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;values:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- hybrid&nbsp;&nbsp;
&nbsp;&nbsp;
# IPAM configuration for pod networking
ipam:
&nbsp;&nbsp;mode: cluster-pool
&nbsp;&nbsp;operator:
&nbsp;&nbsp; &nbsp;clusterPoolIPv4PodCIDRList:
&nbsp;&nbsp; &nbsp;- 192.168.64.0/24&nbsp; &nbsp;&nbsp;# RemotePodNetwork CIDR
&nbsp;&nbsp; &nbsp;clusterPoolIPv4MaskSize: 25

# Cilium Operator configuration
operator:
&nbsp;&nbsp;rollOutPods: true
&nbsp;&nbsp;unmanagedPodWatcher:
&nbsp;&nbsp; &nbsp;restart: false
&nbsp;&nbsp;affinity:
&nbsp;&nbsp; &nbsp;nodeAffinity:
&nbsp;&nbsp; &nbsp; &nbsp;requiredDuringSchedulingIgnoredDuringExecution:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;nodeSelectorTerms:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;- matchExpressions:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- key: eks.amazonaws.com/compute-type
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;operator: In
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;values:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- hybrid</code></pre> 
 </div> 
 <ol start="2"> 
  <li>Install Cilium on EKS Hybrid Nodes using Helm with the preceding configuration.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">helm repo add cilium https://helm.cilium.io/
CILIUM_VERSION=1.18.6

helm install cilium cilium/cilium \
--version ${CILIUM_VERSION} \
--values cilium-values.yaml \
--namespace kube-system</code></pre> 
 </div> 
 <ol start="3"> 
  <li>If you’re running <a href="https://kubernetes.io/docs/reference/access-authn-authz/webhook/" rel="noopener noreferrer" target="_blank">webhooks</a> on hybrid nodes, then you must make sure that on-premises Pod CIDRs are routable across the hybrid network environment, using techniques such as BGP routing, static routing, or <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-concepts-kubernetes.html#hybrid-nodes-concepts-k8s-pod-cidrs" rel="noopener noreferrer" target="_blank">ARP proxying</a>. This demo uses <a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cilium-bgp.html" rel="noopener noreferrer" target="_blank">Cilium BGP control-plane</a> to enable BGP peering between hybrid nodes and on-premises routers, and to advertise Pod CIDRs to the on-premises network.</li> 
 </ol> 
 <p>Apply the following Cilium BGP configuration to your cluster.</p> 
 <div class="hide-language"> 
  <pre><code class="lang-yaml">---
apiVersion: cilium.io/v2
kind: CiliumBGPClusterConfig
metadata:
&nbsp;&nbsp;name: cilium-bgp
spec:
&nbsp;&nbsp;nodeSelector:
&nbsp;&nbsp; &nbsp;matchExpressions:
&nbsp;&nbsp; &nbsp;- key: eks.amazonaws.com/compute-type
&nbsp;&nbsp; &nbsp; &nbsp;operator: In
&nbsp;&nbsp; &nbsp; &nbsp;values:
&nbsp;&nbsp; &nbsp; &nbsp;- hybrid
&nbsp;&nbsp;bgpInstances:
&nbsp;&nbsp;- name: "cilium-bgp"
&nbsp;&nbsp; &nbsp;localASN: &lt;NODES_ASN&gt;
&nbsp;&nbsp; &nbsp;peers:
&nbsp;&nbsp; &nbsp;- name: "onprem-router"
&nbsp;&nbsp; &nbsp; &nbsp;peerASN: &lt;ONPREM_ROUTER_ASN&gt;
&nbsp;&nbsp; &nbsp; &nbsp;peerAddress: &lt;ONPREM_ROUTER_IP&gt;
&nbsp;&nbsp; &nbsp; &nbsp;peerConfigRef:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;name: "cilium-peer"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
---
apiVersion: cilium.io/v2
kind: CiliumBGPPeerConfig
metadata:
&nbsp;&nbsp;name: cilium-peer
spec:
&nbsp;&nbsp;timers:
&nbsp;&nbsp; &nbsp;holdTimeSeconds: 30
&nbsp;&nbsp; &nbsp;keepAliveTimeSeconds: 10
&nbsp;&nbsp;gracefulRestart:
&nbsp;&nbsp; &nbsp;enabled: true
&nbsp;&nbsp; &nbsp;restartTimeSeconds: 120
&nbsp;&nbsp;families:
&nbsp;&nbsp; &nbsp;- afi: ipv4
&nbsp;&nbsp; &nbsp; &nbsp;safi: unicast
&nbsp;&nbsp; &nbsp; &nbsp;advertisements:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;matchLabels:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;advertise: "bgp"

---
apiVersion: cilium.io/v2
kind: CiliumBGPAdvertisement
metadata:
  name: bgp-adv-pod
  labels:
    advertise: bgp
spec:
  advertisements:
    - advertisementType: "PodCIDR"</code></pre> 
 </div> 
 <ol start="4"> 
  <li>Validate that your nodes are connected to the EKS cluster and in a <code>Ready</code> state.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">$ kubectl get nodes -o wide -l eks.amazonaws.com/compute-type=hybrid
NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; STATUS &nbsp; ROLES &nbsp; &nbsp;AGE &nbsp; VERSION &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; INTERNAL-IP &nbsp; &nbsp; &nbsp; EXTERNAL-IP &nbsp; OS-IMAGE &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KERNEL-VERSION &nbsp; &nbsp; &nbsp; CONTAINER-RUNTIME
mi-0e06d30895cfcc155 &nbsp; Ready &nbsp; &nbsp;&lt;none&gt; &nbsp; 17d &nbsp; v1.34.2-eks-ecaa3a6 &nbsp; 192.168.100.101 &nbsp; &lt;none&gt; &nbsp; &nbsp; &nbsp; &nbsp;Ubuntu 24.04.3 LTS &nbsp; 6.14.0-1015-nvidia &nbsp; containerd://2.2.1</code></pre> 
 </div> 
 <h3>Install NVIDIA GPU Operator for Kubernetes</h3> 
 <p>The NVIDIA GPU Operator uses the&nbsp;Kubernetes&nbsp;<a href="https://www.cncf.io/projects/operator-framework/" rel="noopener noreferrer" target="_blank">operator framework</a> to automate the lifecycle management of NVIDIA software components required to provision GPU resources. These components include the NVIDIA drivers (for enabling <a href="https://developer.nvidia.com/cuda/" rel="noopener noreferrer" target="_blank">CUDA</a>), Kubernetes device plugin for GPUs, the <a href="https://github.com/NVIDIA/nvidia-container-toolkit" rel="noopener noreferrer" target="_blank">NVIDIA Container Toolkit</a>, and&nbsp;DCGM&nbsp;based monitoring and others.</p> 
 <ol> 
  <li>Deploy NVIDIA GPU Operator on hybrid nodes using the official Helm chart.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update

helm install gpu-operator nvidia/gpu-operator \
--namespace gpu-operator \
--create-namespace \
--set driver.enabled=true&nbsp;\
--set toolkit.enabled=true \
--set devicePlugin.enabled=true \
--set gfd.enabled=true \
--set migManager.enabled=true \
--set nodeStatusExporter.enabled=true \
--set dcgmExporter.enabled=true \
--set operator.defaultRuntime=containerd \
--set operator.runtimeClass=nvidia \
--wait</code></pre> 
 </div> 
 <ol start="2"> 
  <li>Wait until all pods in the&nbsp;<code>gpu-operator</code> namespace are running or completed.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">$ kubectl get pods -n gpu-operator
NAMESPACE &nbsp; &nbsp; &nbsp;NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY &nbsp; STATUS &nbsp; &nbsp; &nbsp;RESTARTS &nbsp; &nbsp; &nbsp; &nbsp; AGE
gpu-operator &nbsp; gpu-feature-discovery-7jvph &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; gpu-operator-7569f8b499-7k59n &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; &nbsp;27m
gpu-operator &nbsp; gpu-operator-node-feature-discovery-gc-55ffc49ccc-glq9l &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; &nbsp;27m
gpu-operator &nbsp; gpu-operator-node-feature-discovery-master-6b5787f695-n92x4 &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; &nbsp;27m
gpu-operator &nbsp; gpu-operator-node-feature-discovery-worker-9wqq5 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; nvidia-container-toolkit-daemonset-f9brm &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; nvidia-cuda-validator-nzwmh &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0/1 &nbsp; &nbsp; Completed &nbsp; 0 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;92s
gpu-operator &nbsp; nvidia-dcgm-exporter-hn4vz &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; nvidia-device-plugin-daemonset-4kb5c &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; nvidia-node-status-exporter-xpz9j &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d
gpu-operator &nbsp; nvidia-operator-validator-t662d &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; &nbsp; 1 (2m39s ago) &nbsp; 15d</code></pre> 
 </div> 
 <ol start="3"> 
  <li>The NVIDIA GPU Operator validates the stack using the <code>nvidia-operator-validator</code> and the <code>nvidia-cuda-validator</code> pods. Verify the logs on these pods and confirm that the validations are successful.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">$ kubectl logs -n gpu-operator nvidia-operator-validator-t662d
Defaulted container "nvidia-operator-validator" out of: nvidia-operator-validator, driver-validation (init), toolkit-validation (init), cuda-validation (init), plugin-validation (init)
all validations are successful

$ kubectl logs -n gpu-operator nvidia-cuda-validator-nzwmh
Defaulted container "nvidia-cuda-validator" out of: nvidia-cuda-validator, cuda-validation (init)
cuda workload validation is successful</code></pre> 
 </div> 
 <ol start="4"> 
  <li>The GPU within the DGX Spark node is now exposed to the kubelet and is visible in&nbsp;<a href="https://kubernetes.io/docs/reference/node/node-status/#capacity" rel="noopener noreferrer" target="_blank">nodes allocatable</a>:</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">$ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu" -l eks.amazonaws.com/compute-type=hybrid
NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; GPU
mi-0e06d30895cfcc155 &nbsp; 1</code></pre> 
 </div> 
 <h3>Deploy NVIDIA NIM for inference on EKS Hybrid Nodes</h3> 
 <ol> 
  <li>To deploy NVIDIA NIM, you must set up an&nbsp;<a href="https://docs.nvidia.com/ngc/latest/ngc-catalog-user-guide.html#generating-ngc-api-keys" rel="noopener noreferrer" target="_blank">NVIDIA NGC API key</a> and create container registry secrets using the key.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">kubectl create secret docker-registry ngc-secret --docker-server=nvcr.io --docker-username='$oauthtoken' --docker-password=$NGC_API_KEY
kubectl create secret generic ngc-api --from-literal=NGC_API_KEY=$NGC_API_KEY</code></pre> 
 </div> 
 <ol start="2"> 
  <li>Download the NIM Helm chart using the following command:</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">helm fetch https://helm.ngc.nvidia.com/nim/charts/nim-llm-&lt;version_number&gt;.tgz --username='$oauthtoken' --password=$NGC_API_KEY
cd nim-deploy/helm</code></pre> 
 </div> 
 <ol start="3"> 
  <li>Select a <a href="https://docs.nvidia.com/nim/large-language-models/latest/supported-models.html#optimized-models" rel="noopener noreferrer" target="_blank">supported model</a> for NVIDIA NIM based on the GPU specification of your hybrid nodes. Create the helm charts overrides using the NIM container image path, and set the&nbsp;<code>ngcAPISecret</code> and&nbsp;<code>imagePullSecrets</code>&nbsp;using the secrets created in Step 1.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-yaml">cat &gt; qwen3-32b-spark-nim.values.yaml &lt;&lt;EOF
image:
    repository: "nvcr.io/nim/qwen/qwen3-32b-dgx-spark"
    tag: 1.0.0-variant
model:
  ngcAPISecret: ngc-api
nodeSelector:
  eks.amazonaws.com/compute-type: hybrid
resources:
  limits:
    nvidia.com/gpu: 1
persistence:
  enabled: false
imagePullSecrets:
  - name: ngc-secret
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
EOF</code></pre> 
 </div> 
 <ol start="4"> 
  <li>Deploy a NIM based LLM using the following command. In this example I’m running a <a href="https://catalog.ngc.nvidia.com/orgs/nim/teams/qwen/containers/qwen3-32b-dgx-spark?version=1.1.0-variant" rel="noopener noreferrer" target="_blank">Qwen3-32B</a> image that is specifically optimized for the DGX Spark node.</li> 
 </ol> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">helm install my-nim nim-llm-1.15.4.tgz -f ./qwen3-32b-spark-nim.values.yaml</code></pre> 
  <p>This deployment isn’t persistent and doesn’t use a model cache. To implement a model cache, you need to install CSI drivers and configure <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/" rel="noopener noreferrer" target="_blank">Persistent Volumes</a> using the on-premises storage infrastructure.</p> 
  <ol start="5"> 
   <li>The NIM pod deployed on hybrid nodes is routable through BGP, thus you can directly access its API to test the model.</li> 
  </ol> 
  <div class="hide-language"> 
   <pre><code class="lang-css">$ kubectl get pods -o wide | grep nim
my-nim-nim-llm-0 &nbsp; 1/1 &nbsp; &nbsp; Running &nbsp; 0 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;86m &nbsp; 192.168.64.102 &nbsp; mi-0e06d30895cfcc155 &nbsp; &lt;none&gt; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;none&gt;

$ curl -X 'POST' \
&nbsp;&nbsp;"http://192.168.64.102:8000/v1/chat/completions" \
&nbsp;&nbsp;-H 'accept: application/json' \
&nbsp;&nbsp;-H 'Content-Type: application/json' \
&nbsp;&nbsp;-d '{
&nbsp;&nbsp; &nbsp; &nbsp;"model": "Qwen/Qwen3-32B",
&nbsp;&nbsp; &nbsp; &nbsp;"prompt": "What is Kubernetes?",
&nbsp;&nbsp; &nbsp; &nbsp;"max_tokens": 100
&nbsp;&nbsp; &nbsp; &nbsp;}'</code></pre> 
  </div> 
  <p>The following is an example of expected response:</p> 
  <div class="hide-language"> 
   <pre><code class="lang-css">{
&nbsp;&nbsp;"id": "cmpl-d5161978bda9401b9b7a4ef0a529b6ce",
&nbsp;&nbsp;"object": "text_completion",
&nbsp;&nbsp;"created": 1770465499,
&nbsp;&nbsp;"model": "Qwen/Qwen3-32B",
&nbsp;&nbsp;"choices": [
&nbsp;&nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp;"index": 0,
&nbsp;&nbsp; &nbsp; &nbsp;"text": " Why do you need it?\n\nKubernetes is a container orchestration system that automates the deployment, scaling, and management of containerized applications. It is an open-source system that was originally developed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF). Kubernetes allows developers to easily deploy and manage applications in a distributed environment, making it a popular choice for organizations that use containerized applications.\n\nOne of the main reasons why Kubernetes is needed is because it provides a way to manage container",
&nbsp;&nbsp; &nbsp; &nbsp;"logprobs": null,
&nbsp;&nbsp; &nbsp; &nbsp;"finish_reason": "length",
&nbsp;&nbsp; &nbsp; &nbsp;"stop_reason": null,
&nbsp;&nbsp; &nbsp; &nbsp;"prompt_logprobs": null
&nbsp;&nbsp; &nbsp;}
&nbsp;&nbsp;],
&nbsp;&nbsp;"service_tier": null,
&nbsp;&nbsp;"system_fingerprint": null,
&nbsp;&nbsp;"usage": {
&nbsp;&nbsp; &nbsp;"prompt_tokens": 4,
&nbsp;&nbsp; &nbsp;"total_tokens": 104,
&nbsp;&nbsp; &nbsp;"completion_tokens": 100,
&nbsp;&nbsp; &nbsp;"prompt_tokens_details": null
&nbsp;&nbsp;},
&nbsp;&nbsp;"kv_transfer_params": null
}</code></pre> 
  </div> 
  <p>You have successfully deployed an LLM using NVIDIA NIM on your EKS Hybrid Nodes.</p> 
  <h3>Configure centralized monitoring and observability for GPU metrics</h3> 
  <p>The following two sections walk you through configuring centralized monitoring and observability for GPU metrics.</p> 
  <h4>Install EKS Node Monitoring Agent</h4> 
  <p>The EKS Node Monitoring Agent (NMA) is bundled into a container image that can be deployed as a DaemonSet across your EKS Hybrid Nodes. It&nbsp;collects node health information and detects GPU-specific issues using the NVIDIA DCGM and <a href="https://developer.nvidia.com/management-library-nvml" rel="noopener noreferrer" target="_blank">NVIDIA Management Library (NVML)</a>. It reports health issues by&nbsp;updating node status conditions and emitting Kubernetes events. Go to this <a href="https://aws.amazon.com/blogs/containers/amazon-eks-introduces-node-monitoring-and-auto-repair-capabilities/" rel="noopener noreferrer" target="_blank">AWS Container post</a> to learn more details on NMA.</p> 
  <ol> 
   <li>To install the NMA on hybrid nodes, use the following AWS CLI command to create the Amazon EKS add-on.</li> 
  </ol> 
  <div class="hide-language"> 
   <pre><code class="lang-bash">aws eks create-addon --cluster-name &lt;CLUSTER_NAME&gt; --addon-name eks-node-monitoring-agent</code></pre> 
   <ol start="2"> 
    <li>When it’s installed, NMA starts collecting&nbsp;<a href="https://docs.aws.amazon.com/eks/latest/userguide/node-health.html#node-monitoring-agent" rel="noopener noreferrer" target="_blank">custom node conditions</a> for the EKS Hybrid Nodes. From the following example, you can see NMA detected the 200 GbE clustering interface (enp1s0f0np0) of the hybrid node is disconnected because I am only using a single DGX Spark.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-bash">kubectl describe node mi-0e06d30895cfcc155 | sed -n '/^Conditions:/,/^Addresses:/p' | head -n -1
Conditions:
&nbsp;&nbsp;Type &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Status &nbsp;LastHeartbeatTime &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; LastTransitionTime &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Reason &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Message
&nbsp;&nbsp;---- &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ------ &nbsp;----------------- &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ------------------ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;------ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; -------
&nbsp;&nbsp;NetworkingReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;False &nbsp; Sat, 07 Feb 2026 23:52:59 +1100 &nbsp; Sat, 07 Feb 2026 05:22:59 +1100 &nbsp; InterfaceNotRunning &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Interface Name: "enp1s0f0np0", MAC: "4c:bb:47:2c:11:1d" is not up
&nbsp;&nbsp;KernelReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;True &nbsp; &nbsp;Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; KernelIsReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Monitoring for the Kernel system is active
&nbsp;&nbsp;AcceleratedHardwareReady &nbsp; True &nbsp; &nbsp;Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; NvidiaAcceleratedHardwareIsReady &nbsp; Monitoring for the Nvidia AcceleratedHardware system is active
&nbsp;&nbsp;ContainerRuntimeReady &nbsp; &nbsp; &nbsp;True &nbsp; &nbsp;Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; ContainerRuntimeIsReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Monitoring for the ContainerRuntime system is active
&nbsp;&nbsp;StorageReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; True &nbsp; &nbsp;Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; Sat, 07 Feb 2026 05:12:28 +1100 &nbsp; DiskIsReady &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Monitoring for the Disk system is active
&nbsp; [...]</code></pre> 
   </div> 
   <ol start="3"> 
    <li>NMA also provides an automated log collection method through a Kubernetes CRD called <code>NodeDiagnostic</code>. To enable the log collection from your hybrid nodes, create a <code>NodeDiagnostic</code> custom resource on your cluster, and refer to the Amazon <a href="https://docs.aws.amazon.com/eks/latest/userguide/auto-get-logs.html" rel="noopener noreferrer" target="_blank">EKS user guide</a> for more details.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-yaml">apiVersion: eks.amazonaws.com/v1alpha1
kind: NodeDiagnostic
metadata:
&nbsp;&nbsp;name: &lt;HYBRID_NODE_NAME&gt;
spec:
&nbsp;&nbsp;logCapture:
&nbsp;&nbsp; &nbsp;destination:&nbsp;&lt;S3_PRESIGNED_HTTP_PUT_URL&gt;</code></pre> 
   </div> 
   <h4>Integrate NVIDIA DCGM Exporter with&nbsp;Amazon Managed Service for Prometheus and&nbsp;Amazon Managed Grafana</h4> 
   <p>Beyond node health monitoring, you can use the NVIDIA DCGM Exporter (within the GPU Operator stack) to gather GPU performance metrics and telemetry data that can be scraped by Prometheus. This section shows how to integrate DCGM Exporter with Amazon Managed Service for Prometheus and Amazon Managed Grafana to enable enhanced GPU observability across your EKS Hybrid Nodes.</p> 
   <ol> 
    <li>Start by creating an Amazon Managed Service for Prometheus workspace.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-bash">aws amp create-workspace --alias dgx-spark-metrics --region ap-southeast-2 --query 'workspaceId' --output text&nbsp;</code></pre> 
   </div> 
   <ol start="2"> 
    <li>Next, follow <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-OpenTelemetry.html" rel="noopener noreferrer" target="_blank">this user guide</a> to create an IAM role that allows Prometheus to&nbsp;ingest the scraped GPU metrics from EKS Hybrid Nodes to the managed workspace. Verify that the role has the following permissions attached.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-json">{
&nbsp;&nbsp; &nbsp;"Version":"2012-10-17", &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 
&nbsp;&nbsp; &nbsp;"Statement": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"aps:RemoteWrite",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"aps:GetSeries",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"aps:GetLabels",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"aps:GetMetricMetadata"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"Resource": "*"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;]
}</code></pre> 
   </div> 
   <ol start="3"> 
    <li>Prepare a Prometheus installation Helm values file as the following example. Provide the Prometheus ingestion role <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html" rel="noopener noreferrer" target="_blank">Amazon Resource Name (ARN)</a> from the last step, update the <code>remoteWrite</code>&nbsp;endpoint path with the&nbsp;managed Prometheus workspace URL, and add the DCGM Exporter scrape configurations.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-yaml"># RBAC permissions for service discovery
rbac:
&nbsp;&nbsp;create: true

serviceAccounts:
&nbsp;&nbsp;server:
&nbsp;&nbsp; &nbsp;name: amp-iamproxy-ingest-service-account
&nbsp;&nbsp; &nbsp;annotations: 
&nbsp;&nbsp; &nbsp; &nbsp;eks.amazonaws.com/role-arn:&nbsp;&lt;AMP-INGEST-ROLE-ARN&gt;

server:
&nbsp;&nbsp;persistentVolume:
&nbsp;&nbsp; &nbsp;enabled: false
&nbsp;&nbsp;remoteWrite:
&nbsp;&nbsp; &nbsp;- url: https://&lt;AWS-Managed-Prometheus-Workspace-URL&gt;/api/v1/remote_write
&nbsp;&nbsp; &nbsp; &nbsp;sigv4:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;region:&nbsp;&lt;CLUSTER_REGION&gt;
&nbsp;&nbsp; &nbsp; &nbsp;queue_config:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;max_samples_per_send: 1000
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;max_shards: 200
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;capacity: 2500
&nbsp;&nbsp;global:
&nbsp;&nbsp; &nbsp;scrape_interval: 30s
&nbsp;&nbsp; &nbsp;external_labels:
&nbsp;&nbsp; &nbsp; &nbsp;cluster:&nbsp;&lt;CLUSTER_NAME&gt;

# Additional scrape configs for DCGM Exporter
serverFiles:
&nbsp;&nbsp;prometheus.yml:
&nbsp;&nbsp; &nbsp;scrape_configs:
&nbsp;&nbsp; &nbsp; &nbsp;# DCGM Exporter - GPU metrics
&nbsp;&nbsp; &nbsp; &nbsp;- job_name: 'dcgm-exporter'
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;kubernetes_sd_configs:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- role: endpoints&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;# Auto-discover Kubernetes endpoints
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;namespaces:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;names:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- gpu-operator&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;# Look in gpu-operator namespace
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;relabel_configs:
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- source_labels: [__meta_kubernetes_service_name]
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;regex: nvidia-dcgm-exporter&nbsp; &nbsp;&nbsp;# Match the DCGM exporter service
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;action: keep
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;- source_labels: [__meta_kubernetes_pod_node_name]
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;target_label: node&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;# Add node label to metrics</code></pre> 
   </div> 
   <ol start="4"> 
    <li>Use Helm to deploy Prometheus to hybrid nodes using the preceding values. Prometheus uses&nbsp;DCGM Exporter to scrape GPU performance metrics and remote write to the Amazon Managed Service for Prometheus workspace.</li> 
   </ol> 
   <div class="hide-language"> 
    <pre><code class="lang-bash">helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
helm repo update

kubectl create namespace prometheus

helm install prometheus prometheus-community/prometheus \
&nbsp;&nbsp;-n prometheus \
&nbsp;&nbsp;-f ./prometheus-amp-helm-values.yaml</code></pre> 
   </div> 
   <ol start="5"> 
    <li>Follow <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html" rel="noopener noreferrer" target="_blank">this guide</a> to create an Amazon Managed Grafana workspace, including the necessary permissions and authentication access through the IAM Identity Center. Then, configure the Grafana workspace to <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMP-adding-AWS-config.html" rel="noopener noreferrer" target="_blank">add Amazon Managed Service for Prometheus as a data source</a>.</li> 
    <li>Finally, <a href="https://docs.aws.amazon.com/grafana/latest/userguide/getting-started-grafanaui.html" rel="noopener noreferrer" target="_blank">create a new Grafana dashboard</a> (or import one <a href="https://grafana.com/grafana/dashboards/22515-nvidia-dcgm-dashboard/" rel="noopener noreferrer" target="_blank">like&nbsp;this</a>) to visualize scraped GPU metrics such as GPU utilization, GPU memory used, and GPU temperature and energy consumption.</li> 
   </ol> 
   <div class="wp-caption alignnone" id="wp-image-20276" style="width: 2948px;">
    <a href="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-image-2.png"><img alt="Figure 2: Use Amazon Managed Grafana to monitor and visualize GPU metrics and telemetry across hybrid nodes" class="size-full wp-image-20276" height="2069" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-image-2.png" width="2938" /></a>
    <p class="wp-caption-text" id="caption-wp-image-20276">Figure 2: Use Amazon Managed Grafana to monitor and visualize GPU metrics and telemetry across hybrid nodes</p>
   </div> 
   <p>You can integrate EKS Hybrid Nodes with AWS cloud services to streamline generative AI deployment on-premises by removing the Kubernetes management overhead, while maintaining consistent operational practices with centralized observability across cloud, on-premises, and edge locations.</p> 
   <h2>Cleaning up</h2> 
   <p>To avoid incurring long-term charges, delete the AWS resources created as part of the demo walkthrough.</p> 
   <div class="hide-language"> 
    <pre><code class="lang-bash">helm delete my-nim
helm delete prometheus -n prometheus
aws amp delete-workspace --workspace-id &lt;AMP-WORKSPACE-ID&gt; --region &lt;AWS_REGION&gt;
aws grafana delete-workspace --workspace-id &lt;AMG-WORKSPACE-ID&gt; --region &lt;AWS_REGION&gt;
eksctl delete cluster --name &lt;CLUSTER_NAME&gt; --region &lt;CLUSTER_REGION&gt;</code></pre> 
   </div> 
   <p>Clean up other prerequisite resources that you created if they’re no longer needed.</p> 
   <h2>Conclusion</h2> 
   <p>This post provides a practical example of how Amazon EKS Hybrid Nodes empowers generative AI deployment using your own GPU nodes at on-premises and edge locations.&nbsp;Organizations can use EKS Hybrid Nodes to accelerate AI implementation with data locality and minimal latency, while maintaining consistent management and centralized observability across distributed environments.</p> 
   <p>To learn more about EKS Hybrid Nodes or running AI/ML workloads on Amazon EKS, explore the following resources:</p> 
   <ul> 
    <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-overview.html" rel="noopener noreferrer" target="_blank">EKS Hybrid Nodes user guide</a></li> 
    <li><a href="https://aws.amazon.com/blogs/containers/a-deep-dive-into-amazon-eks-hybrid-nodes/" rel="noopener noreferrer" target="_blank">AWS Blog: A deep dive into Amazon EKS Hybrid Nodes</a></li> 
    <li><a href="https://www.youtube.com/watch?v=ZxC7SkemxvU" rel="noopener noreferrer" target="_blank">AWS re:Invent 2024 session (KUB205) – Bring the power of Amazon EKS to your on-premises applications&nbsp;</a></li> 
    <li><a href="https://awslabs.github.io/ai-on-eks/" rel="noopener noreferrer" target="_blank">AWS AI on EKS project&nbsp;</a></li> 
   </ul> 
   <hr style="width: 80%; margin-top: 2em;" /> 
   <h2>About the authors</h2> 
   <footer> 
    <div class="blog-author-box"> 
     <h3 class="lb-h4"><a href="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-bio_shengchn.jpg"><img alt="" class="size-full wp-image-20280 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/06/CONTAINERS-184-bio_shengchn.jpg" width="100" /></a></h3> 
     <p><strong>Sheng Chen</strong> is a Sr. Specialist Solutions Architect at AWS Australia, bringing over 20 years of experience in IT infrastructure, cloud architecture, and multi-cloud networking. In his current role, Sheng helps customers accelerate cloud migrations and infrastructure modernization by leveraging cloud-native technologies. He specializes in Amazon EKS, AWS hybrid cloud services, platform engineering and AI infrastructure.</p> 
    </div> 
    <div class="blog-author-box"> 
     <h3 class="lb-h4"><a href="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/11/CONTAINERS-184-bio_erchpm.jpeg"><img alt="" class="size-full wp-image-20310 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2026/03/11/CONTAINERS-184-bio_erchpm.jpeg" width="100" /></a></h3> 
     <p><strong>Eric Chapman</strong> is a Product Manager Technical at AWS. He focuses on bringing the power of Amazon EKS to wherever customers need to run their Kubernetes workloads.</p> 
    </div> 
   </footer> 
  </div> 
 </div> 
</div>
