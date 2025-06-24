# EKS Cluster (EC2-based Workers) via Terragrunt

This directory provisions an [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/) cluster backed by **EC2 instances**, using a Terragrunt wrapper around a customized [Gruntwork Terraform module](https://github.com/gruntwork-io/terraform-aws-service-catalog/tree/main/modules/services/eks-cluster).

## 📌 Overview

This Terragrunt configuration deploys:

1. An **EKS control plane**.
2. One or more **EKS Managed Node Groups** (EC2 worker nodes).
3. Associated **IAM roles**, **security groups**, and **network configuration**.
4. Optional **EBS CSI Driver**, **CloudWatch log groups**, and **KMS-based envelope encryption**.

The infrastructure follows Gruntwork best practices and separates environment-specific logic from shared configurations.

---

## 📁 Directory Structure

- `terragrunt.hcl`: Environment-specific overrides (e.g., dev, stage, prod) for the EKS cluster.
- `_envcommon/services/eks-cluster.hcl`: Shared EKS settings across all environments (e.g., VPC dependencies, IAM role mappings, hooks).
- `common.hcl`: Global variables (account IDs, CIDR allowlists, repo URIs, etc.).

---

## 🧩 How It Works

- This module wraps the Terraform code at:
  
  ```hcl
  git::git@github.com:gruntwork-io/terraform-aws-service-catalog.git//modules/services/eks-cluster?ref=v0.119.1
