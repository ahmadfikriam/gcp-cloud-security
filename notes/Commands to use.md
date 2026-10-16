Weeks 1-2: Command Notes
Project ID: `cloud-sec-ahmadfikriam`
Region: `asia-southeast1` (Singapore)
Tool: Cloud Shell
---
1. Setup and verification
Point Cloud Shell at the project and confirm the config:
```bash
gcloud config set project cloud-sec-ahmadfikriam
gcloud config list
```
Enable the APIs for Week 1 (Compute, IAM, Resource Manager):
```bash
gcloud services enable compute.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com
```
---
2. Identity: service account with no roles
Create a dedicated runtime identity. It has no roles, so it can do nothing beyond running the app.
```bash
gcloud iam service-accounts create sa-app-runtime \
  --display-name="App runtime (least privilege)"

gcloud iam service-accounts list
```
---
3. Network: custom VPC
Create a custom-mode VPC and one subnet (new custom VPCs start with no inbound rules):
```bash
gcloud compute networks create secure-vpc --subnet-mode=custom

gcloud compute networks subnets create secure-subnet-sg \
  --network=secure-vpc \
  --region=asia-southeast1 \
  --range=10.10.0.0/24
```
Verify:
```bash
gcloud compute networks list
gcloud compute networks subnets list --network=secure-vpc
```
---
4. Review: what the project allows by default
Firewall rules (shows the permissive `default` network rules for SSH and RDP):
```bash
gcloud compute firewall-rules list --sort-by=NETWORK
```
Project IAM policy (shows the default Compute service account with `roles/editor`):
```bash
gcloud projects get-iam-policy cloud-sec-ahmadfikriam
```
Findings
#	Finding	Risk	Planned fix 
# 01	Default Compute SA has `roles/editor`	A compromised workload inherits broad project access	Use `sa-app-runtime`, then replace Editor with specific roles (Week 3)

# 02	`default` network allows SSH (22) and RDP (3389) from `0.0.0.0/0`	Remote access exposed to the internet	Use `secure-vpc` with IAP-only SSH
---
5. Cloud Run: locked-down deployment
Enable the APIs:
```bash
gcloud services enable run.googleapis.com artifactregistry.googleapis.com cloudbuild.googleapis.com
```
Deploy Google's sample container with the dedicated service account, no public access, and an instance cap:
```bash
gcloud run deploy secure-app \
  --image=us-docker.pkg.dev/cloudrun/container/hello \
  --region=asia-southeast1 \
  --service-account=sa-app-runtime@cloud-sec-ahmadfikriam.iam.gserviceaccount.com \
  --no-allow-unauthenticated \
  --max-instances=2
```
Test authentication. Anonymous should return 403, authenticated should return 200:
```bash
URL=$(gcloud run services describe secure-app --region=asia-southeast1 --format='value(status.url)')
curl -i $URL
curl -i -H "Authorization: Bearer $(gcloud auth print-identity-token)" $URL
```
Confirm there is no public invoker access (output should show only an `etag`, with no `allUsers` or `allAuthenticatedUsers`):
```bash
gcloud run services get-iam-policy secure-app --region=asia-southeast1
```
---
6. Firewall: SSH only through IAP
Allow SSH on `secure-vpc` only from Google's IAP range, not the whole internet:
```bash
gcloud compute firewall-rules create secure-vpc-allow-iap-ssh \
  --network=secure-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20
```
Verify:
```bash
gcloud compute firewall-rules list --filter="network:secure-vpc"
```
Note: no VM exists yet, so this rule is a prepared control.
---
Results summary
`secure-app`: anonymous request returns 403, authenticated request returns 200
Cloud Run runs as `sa-app-runtime`, not the default Compute SA
No public invoker binding on `secure-app`
`secure-vpc` is a custom VPC with a single rule allowing SSH from the IAP range only
Still to do
Replace `roles/editor` on the default Compute SA (Week 3, after testing nothing breaks)
Confirm 2-Step Verification is on for the Google account
Skills Boost labs (IAM, VPC, Audit Logs)
Housekeeping
Redact billing account ID and card digits before uploading any billing screenshots
Never commit keys, tokens or secrets
