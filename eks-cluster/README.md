# EKS Cluster (EC2-based Workers) via Terragrunt

This directory provisions an [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/) cluster using **EC2 instances as worker nodes**, via a Terragrunt wrapper. It leverages a customized version of the [Gruntwork Terraform module](https://github.com/gruntwork-io/terraform-aws-service-catalog/tree/main/modules/services/eks-cluster) for EKS.

---

## 📌 Overview

This module automates the provisioning of:

1. An **EKS control plane**, running in a specified AWS region.
2. One or more **EKS Managed Node Groups**, backed by EC2 instances.
3. **Security Groups**, **IAM roles**, and **subnet associations** for the control plane and workers.
4. Optional integrations such as:
   - **EBS CSI Driver** for dynamic volume provisioning
   - **KMS-based envelope encryption** for secrets
   - **CloudWatch Log Groups** for audit/control plane logs

---

## 📁 Directory Structure

| File / Folder                          | Purpose                                                                 |
|----------------------------------------|-------------------------------------------------------------------------|
| `terragrunt.hcl`                       | Environment-specific config (inputs, node groups, KMS keys, CIDRs)     |
| `_envcommon/services/eks-cluster.hcl` | Shared config for all environments (Terraform source, VPC, IAM, etc.)  |
| `common.hcl`                           | Global variables (account IDs, naming, CIDRs, repo metadata)           |
| `accounts.json`                        | Mapping of environment keys to AWS account IDs                         |

---

## 🧩 terragrunt.hcl (Environment Layer)

This file is environment-specific and performs the following:

- Includes the root `terragrunt.hcl` and `_envcommon/services/eks-cluster.hcl`
- Overrides module version and inputs such as:
  - `kubernetes_version`
  - `managed_node_group_configurations`
  - `ebs_csi_driver_addon_config`
  - `secret_envelope_encryption_kms_key_arn`
  - `endpoint_public_access`, etc.

It also defines dependencies such as the KMS key needed for secret encryption.

---

## 🧩 envcommon/eks-cluster.hcl (Shared Layer)

This shared configuration defines:

- Terraform source location (Gruntwork EKS module)
- IAM role detection logic using `run_cmd`
- Dependency blocks for VPC, bastion host, SNS, and baseline modules
- Common input variables including:
  - Subnet and VPC IDs
  - Control plane and worker configuration
  - IAM-to-RBAC mapping
  - Logging and tagging policies
- `after_hook` logic to:
  - Attach IAM permissions to node group roles
  - Tag EKS cluster security groups (e.g., for Karpenter discovery)

---

## ⚙️ Key Inputs (Examples)

| Input Name                              | Description                                                               |
|----------------------------------------|---------------------------------------------------------------------------|
| `kubernetes_version`                   | Kubernetes control plane version (e.g. `1.32`)                            |
| `managed_node_group_configurations`    | EC2 instance types, count, tags, subnets                                 |
| `ebs_csi_driver_addon_config`          | Optional EBS CSI driver enablement and version                           |
| `eks_cluster_security_group_tags`      | Used by Karpenter for node discovery                                     |
| `secret_envelope_encryption_kms_key_arn` | Points to KMS CMK for secrets encryption                                  |
| `allow_private_api_access_from_cidr_blocks` | CIDRs allowed to access EKS private API                                  |
| `worker_iam_role_arns_for_k8s_role_mapping` | Maps worker IAM roles to Kubernetes RBAC groups                         |

---

## 🔗 Dependencies Used

The module depends on several foundational infrastructure modules, configured via Terragrunt’s `dependency` blocks:

- **VPC**: Supplies subnet IDs and VPC ID
- **KMS**: Provides KMS key ARN used to encrypt secrets
- **Account Baseline**: IAM roles for SSO-based access control
- **Bastion Host**: Security group to allow SSH
- **SNS Topic**: Used for CloudWatch alarm notifications

Each dependency uses `mock_outputs` to support validation and planning locally.

---

## 🛠 Notable Customizations

In addition to the upstream Gruntwork module, this codebase includes customizations made by **AU Small Finance Bank**:

- Centralized configuration for multiple environments (via `_envcommon`)
- CIDR-based allow lists (e.g., GitLab, Cloudflare)
- Dynamic detection of AWS SSO roles and RBAC mapping
- Karpenter support through SG tagging and node role annotations
- Security group injection for SIEM integrations
- Hooks to automate IAM policy attachment and tagging post-apply

---

## 📚 References

- [Gruntwork Terraform AWS Service Catalog](https://github.com/gruntwork-io/terraform-aws-service-catalog)
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Terragrunt Documentation](https://terragrunt.gruntwork.io/docs/)
- [Kubernetes Docs](https://kubernetes.io/docs/home/)

---

## 👥 Contact

For internal questions, onboarding help, or technical assistance, please contact the **Cloud Platform Team** at:

**Email**: psg.platform@aubank.in
