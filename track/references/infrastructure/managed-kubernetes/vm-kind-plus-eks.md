# VM with Kind Cluster + AWS Account + EKS Cluster

Multi-cluster pattern combining a local Kind cluster on a VM with a remote EKS cluster in a sandboxed AWS account. Both clusters are accessible from the VM via separate kubeconfig contexts.

See `cloud-accounts/vm-plus-aws.md` for AWS account configuration details. See `base-compute/single-vm-kind-k3s.md` for Kind setup details.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
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

The VM needs `nested_virtualization: true` and a Docker-ready image for Kind. The VM also needs the AWS CLI and kubectl installed (install in setup if not on the base image).

## Setup Script Pattern

This setup creates both clusters. Kind is fast (~30-60 seconds), but EKS takes ~15 minutes. Start EKS first, create Kind while waiting, then finalize EKS.

```bash
#!/bin/bash

# Install tools on the VM
curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
chmod +x /usr/local/bin/kind

# Install AWS CLI if not on the base image
if ! command -v aws &>/dev/null; then
  curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
  unzip -q awscliv2.zip
  ./aws/install
  rm -rf awscliv2.zip aws/
fi

# Install eksctl
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /usr/local/bin

# Start EKS provisioning in the background (takes ~15 min)
eksctl create cluster \
  --name production \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed &
EKS_PID=$!

# Create local Kind cluster while EKS provisions
kind create cluster --name local --wait 120s

# Rename the Kind context for clarity
kubectl config rename-context kind-local local

# Wait for EKS to finish
wait $EKS_PID

# EKS kubeconfig is merged automatically by eksctl
# Rename the EKS context for clarity
EKS_CONTEXT=$(kubectl config get-contexts -o name | grep production)
kubectl config rename-context "$EKS_CONTEXT" production

# Verify both clusters
kubectl --context local get nodes
kubectl --context production get nodes
```

## Dual Kubeconfig Management

Both clusters share `~/.kube/config` with separate contexts. Teach learners to switch between them:

```bash
# List available contexts
kubectl config get-contexts

# Switch to the local Kind cluster
kubectl config use-context local

# Switch to the remote EKS cluster
kubectl config use-context production

# Run a command against a specific context without switching
kubectl --context production get pods -A
```

## Cleanup Script

```bash
#!/bin/bash

# Delete EKS cluster
eksctl delete cluster --name production --region us-west-2 --wait

# Delete Kind cluster
kind delete cluster --name local
```

## When to Use

- Track teaches multi-cluster scenarios (federation, multi-cluster service mesh, failover).
- Track teaches GitOps with a local development cluster and a remote production cluster.
- Track teaches tools that bridge local and cloud clusters (Loft vCluster, Telepresence, Skupper).
- Track teaches service mesh across clusters (Istio multi-cluster, Linkerd multi-cluster, Consul mesh gateways).
- Track compares local Kubernetes development workflows with cloud deployment.

## When NOT to Use

- Track only needs one Kubernetes cluster -- use `container-plus-eks.md` for EKS or `base-compute/single-vm-kind-k3s.md` for local-only.
- Track does not need a managed cloud cluster -- use `base-compute/single-vm-kind-k3s.md` with multi-node Kind config instead.
- Track needs multiple cloud clusters (two EKS clusters) -- use `container-plus-eks.md` with multiple clusters provisioned in setup.

## Common Pitfalls

- **Resource requirements**. The VM runs Kind (Docker containers acting as Kubernetes nodes) while also holding kubeconfig and tools for EKS. Use at least `n1-standard-4`. If Kind runs multiple nodes or heavy workloads, use `n1-standard-8`.
- **15-minute EKS wait**. Structure the track so early challenges use the Kind cluster while EKS provisions. Or accept the wait and set expectations in track metadata.
- **Context name confusion**. Default context names from Kind (`kind-local`) and EKS (`arn:aws:eks:...`) are unwieldy. Rename them in setup to clear names like `local` and `production` so learners can switch easily.
- **AWS credentials on the VM**. The VM sandbox does not automatically have AWS credentials. Instruqt injects credentials into the sandbox environment, but verify they are available on the VM. Check for `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in the environment.
- **Tool installation**. Unlike the cloud-client container, the VM base image may not include kubectl, helm, eksctl, or the AWS CLI. Install everything needed in setup.
- **Kind networking**. Kind clusters use Docker networking, which is internal to the VM. Services in Kind are not directly reachable from EKS and vice versa. If the track needs cross-cluster communication, set up a service mesh or use `kubectl port-forward`.
- **Parallel provisioning failure**. If the backgrounded EKS creation fails, `wait $EKS_PID` returns a non-zero exit code. Check the exit status and handle failures to avoid leaving learners with a broken setup.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Multi-cluster service mesh with Istio (local + EKS)
- GitOps workflow: develop locally, deploy to EKS with Argo CD
- Kubernetes federation across local and cloud clusters
- Telepresence: bridge local development to a remote EKS cluster
- Multi-cluster observability with a local collector and cloud backend
