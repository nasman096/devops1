Part 2 – Ingress, Egress and Intra-Cluster Communications 
Core Patterns for Microsoft Azure
Public Cloud Infrastructure and Technology
March 2020
Agenda
2
Problem Overview
NGINX as Ingress Controller
Application Gateway as Ingress Controller
Limiting Agent Node Internet Egress
Encrypting Intra-Cluster Communications
Links to Training & Documentation
Azure Kubernetes Service: Part 2 – Ingress, Egress and Intra-Cluster 
Communications
Core Patterns for Microsoft Azure
Azure Kubernetes Service Pattern Series: 
Part 1 – Overview and Private Access
Part 2 – Ingress, Egress and Intra-Cluster Communications
Part 3 – Access Control and Access to On-Premise Resources
3
• AT&T has been using Kubernetes for on-premise workloads for 
several years
• AT&T is migrating Kubernetes-based applications and services to 
Azure
• Azure offers a managed version of Kubernetes – Azure 
Kubernetes Service (AKS)
Problem Overview
Azure Kubernetes Service (AKS)
How are teams that manage their own AKS Clusters expected to configure and operate their 
environments?
• Managed service option makes ownership of Kubernetes cluster 
by application teams easier to operationalize 
(’Easier’ <> ‘Easy’ or ‘Simple’)
• Private Networking and ASPR compliance are required.
• Access to on-premise resources may be required (application 
dependent)
See Part 1 – Overview and Private Access for general description 
of Azure Kubernetes Service
4
• Provides secure, private access for services 
deployed to AKS Worker Nodes. 
• Provisions Standard Load Balancer when 
Ingress Controller is deployed with required 
annotation
• Private Link Service must be created to 
expose Standard Load Balancer to other 
Virtual Networks
• Private Endpoint(s) are required to access 
Private Link Service from other VNet(s).
NGINX as Ingress Controller
Bastion Virtual Network Workload A Virtual Network
Cluster Subnet
Private
Endpoint
Standard Load
Balancer
Private Link 
Service
Bastion Subnet
Conexus
Netbond / 
ExpressRoute
Azure
130.x.x.x 10.x.x.x
CNAME:
130.x.x.55 <fqdn>
NGINX
Client System Ingress Controller
Workload B Virtual Network
Workload B Subnet
10.x.x.x
10.x.x.1
130.x.x.55
AKS Worker
Nodes
annotations:
service.beta.kubernetes.io/azure-load-balancer-internal: "true"
kind: Ingress
metadata:
name: <your-name-here>
namespace: <your-namespace-here>
annotations:
kubernetes.io/ingress.class: nginx
[…]
spec:
rules:
- http:
paths:
- backend:
[…]
5
• * Allows the Azure Application Gateway to be used as the Ingress 
Controller for an Azure Kubernetes cluster (AGIC). 
• Application Gateway Ingress Controller within the AKS cluster as 
a system Pod. 
• Application Gateway Ingress Controller consumes Kubernetes 
Ingress Resources and converts them to an Azure Application 
Gateway configuration.
• Allows Application Gateway to load-balance traffic to Kubernetes 
pods.
• Web Application Firewall feature should also be enabled
• * Private Endpoint should be used to access to Application 
Gateway for internal workloads
• Microsoft Azure OSS: https://azure.github.io/applicationgateway-kubernetes-ingress/
Application Gateway as Ingress Controller
* These features are expected 1H CY2020. Use 
Nginx (or equivalent) Ingress Controller + Azure 
Standard Load Balancer until available.
6
• By Default, AKS clusters have unrestricted outbound internet 
access
• Use Azure Firewall with appropriate rules configured to allow 
required connectivity. When limiting internet egress traffic:
‒ Specific ports and FQDNs needed to maintain AKS must be 
allowed to communicate with required external services
‒ Application needs may require additional rules
• Update route table of AKS subnet, add rule forwarding traffic to 
firewall
Limiting Agent Node Internet Egress
Agent
Nodes
Workload Virtual Network
Cluster Subnet
Azure
Firewall
Internet
FW Subnet 
7
• According to ASPR-0179: Transmission of Electronic Data, all data 
in motion on a CSP network is required to be encrypted
• Encrypting proxy is common feature of Service Mesh product 
offerings. Several well-known products on the market: LinkerD, 
Istio, Consul
• Proxy encrypts communication between services within AKS 
cluster - using a transparent proxy that “wraps” Kubernetes 
service-to-service communication
• * LinkerD is a lightweight service mesh with proxying and routing 
capabilities
• LinkerD is responsible for routing so SvcX -> SvcY can be 
intercepted and rerouted as SvcX -> ProxyX:4141 -> TLS -> 
ProxyY:4140 -> SvcY
• Deployed as a ”sidecar” within every pod on Kubernetes cluster 
(per-service certificates) or as a host-based DaemonSet (clusterwide certificate)
• Distribute certificates as Kubernetes “secrets” to make them 
available to LinkerD proxy instances
Encrypting Intra-Cluster Communications Sidecar Deployment
DaemonSet
Deployment
LinkerD
* LinkerD is a Service Mesh explored by Public 
Cloud Architecture Team, choose a solution that 
best fits your needs.
8
Azure Kubernetes Service
• Azure Kubernetes Service
‒ https://docs.microsoft.com/en-us/azure/aks/
• Using Nginx as Ingress Controller
‒ https://docs.microsoft.com/en-us/azure/aks/ingress-basic
• Create an ingress controller to an internal virtual network in Azure Kubernetes 
Service
‒ https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip
• Create an HTTPS ingress controller and use your own TLS certificates on Azure 
Kubernetes Service
‒ https://docs.microsoft.com/en-us/azure/aks/ingress-own-tls
• AKS Application Gateway as an Ingress Controller
‒ https://azure.github.io/application-gateway-kubernetes-ingress/
Links to Training & Documentation
• Limiting egress traffic for cluster nodes 
‒ https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic
• Azure Firewall Service
‒ https://docs.microsoft.com/en-us/azure/firewall/overview
• ASPR-0179: Transmission of Electronic Data (intra-cluster communications)
‒ https://attegrc.cso.att.com/attegrc/default.aspx?requestUrl=..%2fGenericCon
tent%2fRecord.aspx%3fid%3d4116345%26moduleId%3d370
• LinkerD
‒ https://lnkerd.io/2/overviewatch?v=fVSYaFhMA5Y
‒ https://linkerd.io/2016/10/24/a-service-mesh-for-kubernetes-part-iiiencrypting-all-the-things/
• Architecture Patterns on Public Cloud tSpace Wiki (for supporting materials)
‒ https://wiki.web.att.com/pages/viewpage.action?pageId=1127393658
