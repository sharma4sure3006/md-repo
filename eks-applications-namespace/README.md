# Kubernetes Namespace for Applications (via Terragrunt)

This directory provisions a [Kubernetes Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) within an existing [Amazon EKS](https://aws.amazon.com/eks/) cluster. It uses the `k8s-namespace` module from the [Gruntwork Terraform AWS Service Catalog](https://github.com/gruntwork-io/terraform-aws-service-catalog), wrapped via Terragrunt for consistent multi-environment deployment.

---

## 📌 Overview

This module automates:

1. Creation of a dedicated **namespace** in an EKS cluster (e.g., `applications`).
2. Injection of the correct **Kubernetes provider** and **Helm provider** dynamically using cluster data.
3. Association with upstream dependencies like EKS cluster identity, network, and shared IAM roles.

---

## 📁 Directory Structure

| File / Folder                             | Purpose                                                                 |
|-------------------------------------------|-------------------------------------------------------------------------|
| `terragrunt.hcl`                          | Environment-specific configuration (inputs, dependency to EKS)         |
| `_envcommon/services/eks-applications-namespace.hcl` | Shared config across environments (source URL, provider generation) |
| `common.hcl`                              | Global values like account IDs, naming conventions, repo URLs          |
| `provider_k8s_helm_for_eks.template.hcl`  | Template to dynamically configure `kubernetes` and `helm` providers    |

---

## 🧩 terragrunt.hcl (Environment Layer)

This file is specific to each environment (e.g., `sit`, `prod`) and includes:

- The upstream `_envcommon` config
- The Terraform source version for the module
- A `dependency` block on the `eks-cluster` module to get the cluster name
- Input `name = "applications"` to define the namespace name

---

## 🧩 envcommon/eks-applications-namespace.hcl (Shared Layer)

This configuration provides reusable shared logic for all environments:

- Terraform source location (Gruntwork `k8s-namespace` module)
- Dependency block to pull cluster name from the `eks-cluster` module
- Local values for account, region, and naming standards
- **Provider injection using a `generate` block**, which references:

### ➕ `provider_k8s_helm_for_eks.template.hcl`

This file dynamically configures:

- `kubernetes` provider with:
  - `aws_eks_cluster` data source
  - `kubergrunt` token-based authentication
- `helm` provider with:
  - Same EKS token-based login
  - Authentication to **ECR-based Helm OCI registries** via `aws_ecr_authorization_token`

This ensures seamless and secure Helm deployments into the correct namespace.

---

## ⚙️ Key Inputs (Examples)

| Input Name  | Description                                      |
|-------------|--------------------------------------------------|
| `name`      | Name of the Kubernetes namespace to create       |

---

## 🔗 Dependencies Used

| Dependency     | Purpose                                           |
|----------------|---------------------------------------------------|
| `eks_cluster`  | Retrieves EKS cluster name from the cluster layer |

These dependencies support local development using `mock_outputs` for `plan` and `validate` commands.

---

## 🛠 Notable Customizations

- Namespace is injected with no static provider code — instead it is dynamically generated using Terragrunt’s `generate` block
- EKS cluster name is passed via a Terragrunt dependency on the upstream `eks-cluster`
- `kubergrunt` is used to solve for token expiration and ensure uninterrupted Terraform operations
- Helm OCI authentication is handled natively using AWS ECR authorization tokens

---

## 📚 References

- [Gruntwork Terraform AWS Service Catalog – k8s-namespace module](https://github.com/gruntwork-io/terraform-aws-service-catalog/tree/main/modules/services/k8s-namespace)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Terragrunt Documentation](https://terragrunt.gruntwork.io/docs/)
- [Kubernetes Namespace Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [Kubergrunt by Gruntwork](https://github.com/gruntwork-io/kubergrunt)

---

## 👥 Contact

For internal support or guidance on using this module, please reach out to the **Cloud Platform Team**:

**Email**: psg.platform@aubank.in
