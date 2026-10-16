# GCP Cloud Security Portfolio

Goal: secure a small GCP workload, detect misconfigurations, and automate with Terraform.

## Progress
- [x] Week 1: Project setup, IAM review, custom VPC
- [x] Week 2: Deploy workload on Cloud Run
- [ ] Week 3: Secrets, storage, least privilege
- [ ] Week 4: Logging and detection
- [ ] Week 5: Misconfigure, detect, fix
- [ ] Week 6: Terraform + checkov/tfsec
- [ ] Week 7: Documentation
- [ ] Week 8: Interview prep

## Findings so far
1. Default Compute Engine service account has `roles/editor` (over-privileged).
2. Default network allows SSH (22) and RDP (3389) from anywhere.

## Week 2: Locked-down Cloud Run deployment
- Deployed `secure-app` on Cloud Run (asia-southeast1) with a dedicated service account (`sa-app-runtime`) instead of the default Compute SA.
- Disabled unauthenticated access (`--no-allow-unauthenticated`): anonymous request returns 403, authenticated returns 200.
- Capped `--max-instances=2` to limit blast radius and cost.

## Verified controls
- Custom VPC `secure-vpc` with subnet `10.10.0.0/24` in asia-southeast1, no inbound rules by default
- Cloud Run `secure-app` has no public invoker binding (no allUsers / allAuthenticatedUsers)
- SSH allowed only from the IAP range (35.235.240.0/20), not 0.0.0.0/0
