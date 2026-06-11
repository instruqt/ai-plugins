# Cloud-Client Container + AWS Account + EKS Cluster

Standard pattern for AWS Kubernetes workshops. A cloud-client container provides kubectl, helm, and the AWS CLI. A sandboxed AWS account hosts an EKS cluster provisioned during setup.

See `cloud-accounts/container-plus-aws.md` for AWS account configuration details.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
aws_accounts:
  - name: sandbox
    managed: true
    iam_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "eks:*",
              "ec2:*",
              "ecr:*",
              "elasticloadbalancing:*",
              "autoscaling:*",
              "cloudformation:*",
              "iam:CreateRole",
              "iam:DeleteRole",
              "iam:AttachRolePolicy",
              "iam:DetachRolePolicy",
              "iam:PutRolePolicy",
              "iam:DeleteRolePolicy",
              "iam:GetRole",
              "iam:GetRolePolicy",
              "iam:ListRolePolicies",
              "iam:ListAttachedRolePolicies",
              "iam:PassRole",
              "iam:CreateServiceLinkedRole",
              "iam:CreateInstanceProfile",
              "iam:DeleteInstanceProfile",
              "iam:AddRoleToInstanceProfile",
              "iam:RemoveRoleFromInstanceProfile",
              "iam:GetInstanceProfile",
              "iam:CreateOpenIDConnectProvider",
              "iam:DeleteOpenIDConnectProvider",
              "iam:GetOpenIDConnectProvider",
              "iam:TagOpenIDConnectProvider",
              "sts:GetCallerIdentity",
              "sts:AssumeRole",
              "ssm:GetParameter",
              "kms:CreateKey",
              "kms:DescribeKey",
              "kms:CreateAlias",
              "kms:DeleteAlias",
              "kms:TagResource",
              "logs:CreateLogGroup",
              "logs:DeleteLogGroup",
              "logs:PutRetentionPolicy",
              "logs:DescribeLogGroups",
              "logs:TagResource"
            ],
            "Resource": "*"
          }
        ]
      }
```

## Setup Script Pattern

EKS provisioning takes approximately 15 minutes. The setup script must create the cluster and wait for it to become active before the challenge starts.

### Using eksctl

```bash
#!/bin/bash

# eksctl is pre-installed on cloud-client:2
eksctl create cluster \
  --name lab \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed

# eksctl automatically updates kubeconfig
kubectl get nodes
```

### Using Terraform

```bash
#!/bin/bash

cd /opt/terraform/eks
terraform init
terraform apply -auto-approve

# Retrieve kubeconfig after Terraform completes
aws eks update-kubeconfig --name lab --region us-west-2

kubectl get nodes
```

### Kubeconfig retrieval (if cluster already exists)

```bash
aws eks update-kubeconfig --name lab --region us-west-2
```

## Cleanup Script

EKS clusters are expensive. Always include a cleanup script that destroys the cluster. If the cleanup script fails or is skipped, the sandboxed AWS account still gets destroyed at track end, but leftover resources can delay account cleanup.

```bash
#!/bin/bash

# eksctl cleanup
eksctl delete cluster --name lab --region us-west-2 --wait

# OR Terraform cleanup
cd /opt/terraform/eks
terraform destroy -auto-approve
```

## When to Use

- Track teaches EKS-specific features (managed node groups, Fargate profiles, EKS add-ons, IRSA).
- Track teaches tools that integrate with EKS (AWS Load Balancer Controller, ExternalDNS with Route 53, EBS CSI driver).
- Track teaches Kubernetes on AWS for a customer whose platform is EKS.
- Track needs AWS IAM Roles for Service Accounts (IRSA) or EKS Pod Identity.

## When NOT to Use

- Track teaches generic Kubernetes concepts with no AWS-specific requirements -- use `base-compute/single-vm-kind-k3s.md` instead. It is faster to provision and cheaper to run.
- Track only needs AWS services without Kubernetes -- use `cloud-accounts/container-plus-aws.md` instead.
- Track needs a local development cluster, not a managed cloud cluster -- use `base-compute/single-vm-kind-k3s.md`.

## Common Pitfalls

- **15-minute provisioning time**. EKS cluster creation is slow. This is unavoidable. If the first challenge does not require the cluster, structure the track so early challenges cover non-cluster topics while setup runs. For multi-challenge tracks, provision in the first challenge's setup and verify readiness in the second challenge's setup.
- **IAM policy too narrow**. EKS needs a broad set of IAM permissions across eks, ec2, iam, elasticloadbalancing, autoscaling, and cloudformation. Missing any of these causes cryptic provisioning failures. The policy above is a known-good baseline.
- **Forgetting `iam:PassRole`**. EKS must pass roles to EC2 instances for worker nodes. Without this permission, node group creation fails.
- **Forgetting `iam:CreateServiceLinkedRole`**. The first EKS cluster in an account creates a service-linked role for ELB. Without this permission, the cluster creation hangs waiting for the load balancer role.
- **OIDC provider permissions**. IRSA and EKS Pod Identity require `iam:CreateOpenIDConnectProvider`. Without it, `eksctl utils associate-iam-oidc-provider` fails silently or with an opaque error.
- **Stale kubeconfig**. If the setup script runs `aws eks update-kubeconfig` before the cluster API is ready, the kubeconfig is written but kubectl commands fail with connection refused. Always verify with `kubectl get nodes` after updating kubeconfig.
- **No cleanup script**. EKS clusters cost ~$0.10/hr for the control plane alone, plus EC2 instance costs for worker nodes. Always include a cleanup script. Even though Instruqt destroys the sandbox account, explicit cleanup speeds up account recycling.
- **eksctl version**. The cloud-client image ships a specific eksctl version. If the track needs features from a newer version, install it in setup: `curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /usr/local/bin`.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- EKS cluster provisioning and management with eksctl
- AWS Load Balancer Controller setup and ingress configuration
- IAM Roles for Service Accounts (IRSA) deep dive
- EKS Fargate profile configuration
- Deploying a microservices application on EKS
- EKS add-ons management (CoreDNS, kube-proxy, VPC CNI)
- Kubernetes cluster autoscaling on AWS
