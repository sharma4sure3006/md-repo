# EKS Core Services via Terragrunt

This directory provisions **core Kubernetes services** on an existing [Amazon EKS](https://aws.amazon.com/eks/) cluster using Terragrunt. It leverages a customized version of the [Gruntwork Terraform module](https://github.com/gruntwork-io/terraform-aws-service-catalog/tree/main/modules/services/eks-core-services).

---

## 📌 Overview

This module deploys and configures foundational services essential to the operation and observability of workloads in Kubernetes:

1. **Fluent Bit** – Streams application logs to Amazon CloudWatch Logs or an Elasticsearch-compatible backend.
2. **AWS Load Balancer Controller** – Manages AWS ALBs/NLBs in response to Kubernetes ingress resources.
3. **External DNS** – Automates DNS record management in Route 53 based on Kubernetes annotations.
4. **Cluster Autoscaler** – Adjusts the number of EC2 worker nodes based on pending pod demand.
5. Optional integrations:
   - **CloudWatch Agent**
   - **Private container registry access via ECR**
   - **Custom log routing for SIEM tools**

---

## 📁 Directory Structure

| File / Folder                          | Purpose                                                                 |
|----------------------------------------|-------------------------------------------------------------------------|
| `terragrunt.hcl`                       | Environment-specific config (inputs, overrides, feature toggles)       |
| `_envcommon/services/eks-core-services.hcl` | Shared configuration for this component across all environments   |
| `common.hcl`                           | Common variables (account names, CIDRs, naming conventions, etc.)      |
| `accounts.json`                        | Maps Terragrunt environment names to AWS account IDs                   |

---

## 🧩 terragrunt.hcl (Environment Layer)

This file configures the module for a specific environment (e.g., SIT, UAT, PROD). It includes:

- Source version override (`ref=v0.120.0`)
- Environment-specific inputs:
  - Enable/disable Fluent Bit, Cluster Autoscaler, CloudWatch Agent
  - Fluent Bit customization blocks (`inputs`, `filters`, `outputs`)
  - Ingress controller image versions and ECR registry
- Optional RDS dependency (commented)

It includes `_envcommon` and root configurations and defines critical service dependencies.

---

## 🧩 _envcommon/eks-core-services.hcl (Shared Layer)

This shared file provides core logic across all environments:

- Declares Terraform module source path (`ref=v0.104.14`)
- Hooks to:
  - Login to ECR before applying
  - Patch ALB controller with additional arguments
- Declares dependencies:
  - **VPC** (for subnets)
  - **EKS Cluster**
  - **Applications Namespace**
- Sets up dynamic IAM OIDC and Fargate execution role integration
- Contains `locals` to auto-load region/account/environment metadata

---

## ⚙️ Key Inputs (Examples)

| Input Name                               | Description                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------|
| `enable_fluent_bit`                      | Enables Fluent Bit logging                                                  |
| `fluent_bit_log_group_retention`         | Retention period for CloudWatch log groups                                 |
| `enable_cluster_autoscaler`              | Enables Kubernetes Cluster Autoscaler                                      |
| `schedule_cluster_autoscaler_on_fargate` | Runs autoscaler on Fargate to prevent eviction                             |
| `alb_ingress_controller_chart_version`   | Helm chart version for ALB controller                                      |
| `aws_cloudwatch_agent_version`           | Version of AWS CloudWatch Agent                                            |
| `external_dns_chart_repository_url`      | Helm OCI registry for External DNS                                         |
| `fluent_bit_additional_inputs`           | Custom log input configuration (e.g., tail files)                          |
| `fluent_bit_additional_filters`          | Log filter configuration (e.g., enrich with Kubernetes metadata)           |
| `fluent_bit_additional_outputs`          | Output target (e.g., CloudWatch, Elasticsearch, SIEM)                      |

---

## 🔗 Dependencies Used

This module relies on several upstream modules provisioned via Terragrunt `dependency` blocks:

- **VPC**: Supplies `vpc_id`, private subnet IDs
- **EKS Cluster**: Provides cluster name, OIDC config, Fargate role
- **EKS Applications Namespace**: Supplies the Kubernetes namespace for logging, services, autoscaler
- *(Optional)* RDS dependency for internal DNS mapping (currently commented)

All dependencies are mocked for `plan` and `validate` phases to support isolated local workflows.

---

## 🛠 Notable Customizations

Custom logic added by **AU Small Finance Bank** includes:

- Fluent Bit input/output tailored for **uat-cb** logs and SIEM integrations
- Private ECR integration using `before_hook` to authenticate before Helm deployment
- Dynamic `kubectl patch` for ALB controller to:
  - Disable WAF/WAFv2/Shield via `--enable-*` flags
- Granular input control for logging stack
- CIDR-based access and flexible IAM integration via the shared config

---

## 📚 References

- [Gruntwork Terraform AWS Service Catalog](https://github.com/gruntwork-io/terraform-aws-service-catalog)
- [AWS EKS Observability Guide](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-logs.html)
- [Fluent Bit Documentation](https://docs.fluentbit.io/manual/)
- [Helm Charts for AWS Observability](https://github.com/aws-observability/aws-observability-helm-charts)
- [ExternalDNS Docs](https://github.com/kubernetes-sigs/external-dns)

---

## 👥 Contact

For assistance with logging, observability, or autoscaling configurations, reach out to the **Cloud Platform Team**:

**Email**: psg.platform@aubank.in
