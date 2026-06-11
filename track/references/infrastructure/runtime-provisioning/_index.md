# Runtime Provisioning Reference Index

Runtime provisioning patterns create cloud infrastructure dynamically during sandbox setup using Terraform or cloud CLIs. The sandbox host (typically a cloud-client container) runs IaC tools to provision resources that don't exist until the track starts.

- `terraform-aws-vm.md` — Container provisions a VM in AWS via Terraform at setup time
- `terraform-azure-vm.md` — Container provisions a VM in Azure via Terraform at setup time
- `terraform-gcp-vm.md` — Container provisions a VM in GCP via Terraform or gcloud at setup time (may use IAP tunnels)
