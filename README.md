# GCP Cloud Security Portfolio

Goal: secure a small GCP workload, detect misconfigurations, and automate with Terraform.

## Progress
- [x] Week 1: Project setup, IAM review, custom VPC
- [ ] Week 2: Deploy workload on Cloud Run
- [ ] Week 3: Secrets, storage, least privilege
- [ ] Week 4: Logging and detection
- [ ] Week 5: Misconfigure, detect, fix
- [ ] Week 6: Terraform + checkov/tfsec
- [ ] Week 7: Documentation
- [ ] Week 8: Interview prep

## Findings so far
1. Default Compute Engine service account has `roles/editor` (over-privileged).
2. Default network allows SSH (22) and RDP (3389) from anywhere.
