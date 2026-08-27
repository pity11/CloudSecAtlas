# CloudSecAtlas

A cloud security knowledge and research repository organized around identities, resource relationships, attack paths, detection, and remediation. Concepts are modeled in AWS first, then mapped to Azure, GCP, and Kubernetes.

See [`ROADMAP.md`](ROADMAP.md) for the complete capability map and stage completion criteria.

## Structure

- [`foundations/`](foundations/): shared responsibility, accounts and organizations, control planes, and data planes
- [`iam/`](iam/): policy evaluation, trust relationships, temporary credentials, and privilege-escalation paths
- [`services/`](services/): networking, compute, storage, databases, serverless, KMS, and secrets
- [`containers-k8s/`](containers-k8s/): images, runtimes, ServiceAccounts, RBAC, and workload boundaries
- [`detection-response/`](detection-response/): audit logs, detection, investigation, containment, and recovery
- [`labs/`](labs/): authorized, cost-controlled experiments
- [`references/`](references/): curated external resources and clear tool positioning
- [`templates/`](templates/): standardized experiment templates

## Entry Points

- [`services/`](services/): infrastructure, messaging systems, and identity boundaries
- [`references/`](references/): cloud security knowledge bases, training ranges, and curated public projects

## Core Questions

1. What effective permissions does a principal have, and why?
2. Is a resource exposed, and is the network path actually reachable?
3. Which temporary credentials and secrets can a workload obtain?
4. Which trust relationships form cross-service, cross-role, or cross-account attack paths?
5. Which logs can prove the behavior, and how can containment and recovery be verified?

## Repository Scope

- This repository stores knowledge models, experimental records, attack paths, detection rules, remediation validation, and research questions.
- Mastery requires evidence: the behavior can be explained, reproduced, detected or remediated, and regression-tested.
- Once a topic becomes an independently demonstrable, testable, and evaluable system, it should graduate from CloudSecAtlas into a separate project repository.

## Safety and Cost Controls

- Use only personal accounts, dedicated lab accounts, or explicitly authorized environments.
- Default to least privilege, budget alerts, short-lived resources, and explicit teardown procedures.
- Never commit access keys, session tokens, kubeconfig files, Terraform state, or unsanitized logs.
- Prefer dry runs, plan output, and official or local training ranges for attack simulation.
