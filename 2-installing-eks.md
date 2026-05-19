# Install EKS

Please follow the prerequisites doc before this

## Install using Fargate

```
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

Using command line saves us a lot of time and the hassle of manual configuration

Here we use AWS Fargate as the compute engine:

**AWS Fargate Compute Engine:** Executes pods in separate, hypervisor-isolated compute environments. Each individual pod maps to its own serverless micro-VM instance, eliminating traditional EC2 node footprints.
