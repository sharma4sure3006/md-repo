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

The design separates **environment-specific overrides** from **shared reusable configurations**, adhering to [Gruntwork's Infrastructure as Code best practices](https://gruntwork.io/infrastructure-as-code-library/).

---

## 📁 Directory Structure

| File / Folder | Purpose |
|---------------|---------|
| `terragrunt.hcl` | Environment-specific inputs (e.g., dev, stage, prod), node group configs, encryption keys |
| `_envcommon/services/eks-cluster.hcl` | Shared config for EKS across all environments |
| `common.hcl` | Global variables like account IDs, naming conventions, CIDRs, repo metadata |
| `accounts.json` | Mapping of environment keys to AWS account IDs |
| `networking/vpc`, `_global/account-baseline`, etc. | Terragrunt modules for dependencies (referenced using `dependency` blocks) |

---

## 🧩 How It Works

This code uses **Terragrunt** to orchestrate Terraform module execution with strong environment separation.

### Terraform Source
```hcl
terraform {
  source = "git::git@github.com:gruntwork-io/terraform-aws-service-catalog.git//modules/services/eks-cluster?ref=v0.119.1"
}
