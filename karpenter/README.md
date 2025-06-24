# EKS Karpenter via Terragrunt

This directory provisions [Karpenter](https://karpenter.sh/), an open-source node lifecycle management controller for Kubernetes. It runs on top of **Amazon EKS** and enables **just-in-time node provisioning** based on real-time scheduling needs.

This implementation wraps the [Gruntwork EKS Karpenter module](https://github.com/gruntwork-io/terraform-aws-service-catalog/tree/main/modules/services/eks-karpenter) using Terragrunt.

---

## 📌 Overview

This module sets up the Karpenter controller in an EKS cluster with:

1. Helm-based deployment of the **Karpenter Controller**
2. IAM role assumptions for service account access
3. Integration with **AWS Spot capacity** via ServiceLinkedRole
4. Hooks for:
   - ECR login (for Helm OCI charts)
   - KMS Grant for encrypted secrets
   - Spot ServiceLinkedRole creation

---

## 📁 Directory Structure

| File / Folder                          | Purpose                                                                  |
|----------------------------------------|--------------------------------------------------------------------------|
| `terragrunt.hcl`                       | Environment-specific config (Karpenter chart version, cluster endpoint)  |
| `_envcommon/services/eks-karpenter.hcl` | Shared config for all environments (source, providers, hooks, etc.)      |
| `common.hcl`                           | Global constants like name prefix, account ID map                        |
| `provider_k8s_helm_for_eks.template.hcl` | Generates the Kubernetes and Helm provider blocks for EKS auth          |

---

## 🧩 terragrunt.hcl (Environment Layer)

This environment-specific file performs the following:

- Inherits root and shared configs
- Defines inputs like:
  - `karpenter_chart_version`
  - `eks_cluster_endpoint`
  - `karpenter_chart_additional_values`
- Pulls in the correct EKS cluster details via `dependency` block
- Uses `run_cmd` to fetch live cluster endpoint

---

## 🧩 envcommon/eks-karpenter.hcl (Shared Layer)

This shared config provides:

- Terraform source path for the Gruntwork module
- Private ECR login hook (via `helm registry login`)
- Post-deploy KMS grant creation for controller access
- ServiceLinkedRole hook for Spot instance provisioning
- `k8s_helm_provider.tf` generator to configure providers using a template

---

## ⚙️ Key Inputs (Examples)

| Input Name                            | Description                                                             |
|--------------------------------------|-------------------------------------------------------------------------|
| `karpenter_chart_version`            | Helm chart version for Karpenter (e.g., `0.36.0`)                       |
| `eks_cluster_endpoint`               | Used to connect the controller to the EKS API                          |
| `karpenter_chart_additional_values` | Custom Helm values (e.g., controller image, AWS settings)              |
| `eks_openid_connect_provider_*`     | Needed to configure IAM for Service Accounts (IRSA)                    |

---

## 🔗 Dependencies Used

The following modules are required to deploy Karpenter:

- **VPC**: Supplies private subnets for Karpenter workloads
- **EKS Cluster**: Provides OpenID Connect provider and cluster endpoint
- **IAM / KMS**: Grant-based encryption for secrets via after-hook

Each dependency supports `mock_outputs` for local plan and validation.

---

## 🛠 Notable Customizations

This module is customized for AU Small Finance Bank as follows:

- **Cluster endpoint detection** via `run_cmd` to support dynamic environments
- **KMS key grants** created automatically using `aws kms create-grant`
- **Spot support** via service-linked IAM role hook
- Private ECR registry used for Helm OCI charts

---

## 📚 References

- [Gruntwork Terraform AWS Service Catalog](https://github.com/gruntwork-io/terraform-aws-service-catalog)
- [Karpenter Documentation](https://karpenter.sh/docs/)
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Terragrunt Documentation](https://terragrunt.gruntwork.io/docs/)

---

## 👥 Contact

For internal questions, onboarding help, or technical assistance, please contact the **Cloud Platform Team** at:

**Email**: psg.platform@aubank.in
