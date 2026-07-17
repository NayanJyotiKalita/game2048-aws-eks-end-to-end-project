# End-to-End Guide to Deploying Microservices via the AWS CLI: Going Serverless with EKS Fargate 

For Step by Step Implementation, follow this: </br>
https://medium.com/@nayanjyotikalita_90283/an-end-to-end-guide-to-deploying-microservices-via-the-aws-cli-going-serverless-with-eks-fargate-51ead183df21

Managing production-grade Kubernetes clusters often forces engineering teams into a cycle of maintaining infrastructure, configuring auto-scaling groups, and provisioning underlying EC2 node capacities. 
This project demonstrates how combining Amazon Elastic Kubernetes Service (EKS) with AWS Fargate shifts this dynamic entirely. By eliminating manual node group administration, 
this architecture delivers an automated, serverless compute engine explicitly built for containerized environments. Inside this cluster, we securely deploy a live, 
replicated 2048 game application backed by the AWS Application Load Balancer (ALB) Controller.

## 🏗️ Core Architecture & Components

Before jumping into the terminal, it is critical to understand how the serverless control-plane components interact:  
  - **Amazon EKS Control Plane**: A fully managed, highly available, multi-AZ Kubernetes master node architecture managed by AWS.  
  - **AWS Fargate Compute Engine:** Executes pods in separate, hypervisor-isolated compute environments. Each individual pod maps to its own serverless micro-VM instance, eliminating traditional EC2 node footprints.  
  - **AWS ALB Controller:** Automatically provisions an external AWS Application Load Balancer whenever a Kubernetes Ingress object is declared.  
  - **OIDC Identity Provider:** Establishes a cryptographic trust mapping between AWS IAM and Kubernetes Service Accounts, allowing fine-grained permissions for pods (IRSA) without static credentials.

---

## 🛠️ Prerequisites & Workstation Setup
To provision and operate the infrastructure seamlessly, our deployment workstation requires the following primary binaries:  
  1. kubectl – The Kubernetes cluster CLI tool.
  2. eksctl – The official CLI tool for Amazon EKS.
  3. aws-cli – The Amazon Web Services command-line interface.
  4. helm – The package manager for Kubernetes.
</br>
💡 Step-by-step documentation for installing and configuring these tools locally can be found right here in the Prerequisites Guide [1-prerequisites-installations.md](1-prerequisites-installations.md) 

### **Verification Check**
We confirm our local binary installations execute cleanly without missing dependencies by running:  
```bash
kubectl version --client
eksctl version
aws --version
helm version
```

---

## 🚀 Step-by-Step Deployment Workflow

### 1. Configure AWS CLI & IAM Credentials
We attach an Identity and Access Management (IAM) administrative security credential to map our local workspace authorization. Create an IAM user with `AdministratorAccess` permissions, generate Access Keys , and initialize the profile locally:

```bash
aws configure
# Enter AWS Access Key ID, Secret Access Key, default region (e.g., us-east-1), and output format (json)
```

---

### 2. Provision the Serverless EKS Cluster
Initialize a pure serverless cluster stack configuration using `eksctl`. The presence of the `--fargate` flag explicitly builds the control boundary without creating standard Amazon EC2 worker nodes:

```bash
eksctl create cluster --name <cluster-name> --region us-east-1 --fargate
```

**Note:** This bootstrap sequence automatically configures vital managed system components (EKS Add-ons) including `vpc-cni`, `kube-proxy`, `coredns`, and `metrics-server`. It takes roughly 15-20 minutes to provision the underlying CloudFormation templates. 
</br>
Once created, we synchronize our local system context to point to our new cluster namespace:  
```bash
aws eks update-kubeconfig --name <cluster-name> --region us-east-1
```

---

### 3. Resolve AWS Console RBAC Authorization Issues
When deploying via the CLI, the cluster access maps exclusively to that specific programmatic IAM principal. To expose the resources to console visibility and populate the live dashboard safely, we link the targeted console principal using EKS Access Entries

```bash
# Inject structural cluster authorization linkage
aws eks create-access-entry \
  --cluster-name <cluster-name> \
  --principal-arn arn:aws:iam::<YOUR_ACCOUNT_ID>:root \
  --username root \
  --type STANDARD

# Associate the Cluster Admin Policy
aws eks associate-access-policy \
  --cluster-name <cluster-name> \
  --principal-arn arn:aws:iam::<YOUR_ACCOUNT_ID>:root \
  --policy-arn arn:aws:aws:iam::aws:policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster
```

--

### 4. Configure Multi-Tenant Fargate Profiles
AWS Fargate requires an explicit runtime mapping blueprint, known as a Fargate Profile, to direct Kubernetes namespaces to the serverless engine instead of looking for EC2 instances. Let's isolate our upcoming application into a dedicated namespace profile named `game-2048`

```bash
eksctl create fargateprofile \
  --cluster <cluster-name> \
  --region us-east-1 \
  --name alb-eks-app \
  --namespace game-2048
```

---

### 5. Launch Workloads & Establish the OIDC Trust
Deploy the distributed application manifests (Namespace, Deployment, Service, and Ingress routing components) into the cluster:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

At this phase, running `kubectl get ingress -n game-2048` will reveal an empty `ADDRESS` column. This infrastructure gap exists because vanilla EKS clusters lack an Ingress Controller by default to provision the AWS Application Load Balancer
</br>
To fix this securely, create an OpenID Connect (OIDC) identity provider handshake between EKS and IAM:
```bash
eksctl utils associate-iam-oidc-provider --cluster eks-cluster --approve
```

---

### 6. Install the AWS Load Balancer Controller via Helm
We download the official IAM security schema policy, register it inside our account boundaries, and bind the structural policy directly to a Kubernetes Service Account via an IAM Role (IRSA):

```bash
# Download policy schema
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

# Create IAM policy
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# Bind IAM Policy to Kubernetes Service Account
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

Finally, we deploy the controller chart template engine using Helm, ensuring it points to our native VPC context:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<YOUR_VPC_ID>
```

---

## 🚦 Traffic Route Verification

Once the controller pods initialize successfully, they intercept the lingering ingress object request and spin up a fully managed AWS Application Load Balancer behind the scenes. We run the status query again to verify that our public endpoint address is active:

```bash
kubectl get ingress -n game-2048
```

We will see our public external address route generated under the `ADDRESS` segment:

```
NAME           CLASS   HOSTS   ADDRESS                                                              PORT   AGE
ingress-2048   alb     * k8s-game2048-ingress2-04c19700e8-122068291.us-east-1.elb.amazonaws.com   80     21m
```

Opening this live URL endpoint in any web browser confirms that our external internet request successfully trails directly into our serverless container ecosystem!

---

## 🧼 Infrastructure Resource Deconstruction (Clean-Up)

To avoid running into unexpected or rolling monthly cloud costs, we make sure to dismantle our infrastructure once development testing is completed. Run the commands in this exact sequence to ensure zero stale resource drift:

```bash
# 1. Delete Kubernetes Workloads (Automatically tears down the AWS ALB)
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

# 2. Uninstall AWS Load Balancer Controller Helm chart
helm uninstall aws-load-balancer-controller -n kube-system

# 3. Delete the IAM Service Account stack
eksctl delete iamserviceaccount --cluster eks-cluster --region us-east-1 --namespace kube-system --name aws-load-balancer-controller

# 4. Delete the custom IAM Policy
aws iam delete-policy --policy-arn arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy

# 5. Delete the EKS Cluster and Fargate Profiles completely
eksctl delete cluster --name eks-cluster --region us-east-1
```











