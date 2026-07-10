# ADR-0003: Use Terraform for Cloud Infrastructure

## Status

Accepted

## Context

The platform roadmap includes AWS infrastructure learning and future cloud resources such as networking, IAM, EC2, S3, Route53, and RDS.

The chosen tool should align with the technologies already used professionally and should keep cloud-resource lifecycle management separate from host configuration.

## Decision

Use Terraform to provision and manage cloud infrastructure, primarily in AWS.

Terraform owns:

- VPCs, subnets, route tables, and security groups.
- IAM resources.
- EC2, S3, Route53, RDS, and other future AWS resources.
- Terraform state and backend configuration.
- Cloud-focused learning exercises and reusable modules.

Terraform does not own Fedora host configuration, application deployment internals, or pfSense configuration.

## Rationale

- Terraform is already used in the professional environment.
- It provides directly transferable AWS infrastructure skills.
- It supports declarative, reviewable, and repeatable cloud provisioning.
- It establishes a clean boundary with Ansible.

## Consequences

### Positive

- Strong alignment with current work and career goals.
- Reproducible AWS environments.
- Clear separation between infrastructure provisioning and machine configuration.
- A practical foundation for future CI/CD and Kubernetes infrastructure.

### Negative

- Terraform state must be secured and managed carefully.
- Provider upgrades and state migrations require discipline.
- Cloud experiments can create real costs if lifecycle controls are weak.

## Alternatives Considered

- AWS CloudFormation: native to AWS, but less aligned with the current professional tooling preference.
- AWS CDK: expressive and code-centric, but introduces framework and language coupling before Terraform fundamentals are established.
- Pulumi: attractive for general-purpose languages, but less directly aligned with the current workplace stack.
- Ansible cloud modules: rejected because Ansible should remain focused on machine configuration.

## Related ADRs

- ADR-0002: Use Ansible for Machine Configuration
- ADR-0005: Separate Development, Deployment, and Platform Responsibilities
