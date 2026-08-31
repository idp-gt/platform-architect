# Security

## 1. Authentication with Workload Identity Federation (WIF) on Google Cloud {: #workload-identity }

### What is it and what is it for?

**Workload Identity Federation (WIF)** is an identity management mechanism that allows external workloads (such as GitHub Actions, GitLab CI, AWS, Azure, or On-Premise servers) to authenticate and access Google Cloud resources securely.

Instead of requiring you to store static passwords or JSON files in third-party systems, WIF establishes a trust relationship based on the **OIDC (OpenID Connect)** or SAML protocol. When your CI/CD pipeline needs to make a deployment, it requests a short-lived token from Google Cloud, proving its identity using cryptographic tokens.

### Workload Identity Federation vs. Service Account Keys (JSON Files)

Using static keys (`.json`) is considered an anti-pattern in modern infrastructure security due to the high risks of exfiltration.

| Feature | Service Account Keys (JSON) | Workload Identity Federation (WIF) |
| :--- | :--- | :--- |
| **Lifecycle** | Permanent (until manually revoked or expired). | Temporary (Short-lived tokens, typically 1 hour). |
| **Leak Risk** | Critical. If someone steals the JSON, they have full access from anywhere. | Practically nil. There are no physical secrets to steal. |
| **Operational Overhead** | High. Requires key rotation policies, secure storage, and constant auditing. | Zero. Google and the provider (e.g., GitHub) handle token negotiation automatically. |
| **Access Granularity** | Low. Whoever has the key has all the permissions assigned to the account. | High (ABAC). You can restrict access so that only a specific repository, branch, or *tag* can assume the identity. |

---

### Configuration Guide: GitHub Actions to Google Cloud

Below are the steps to create an OIDC trust tunnel between GitHub and a Google Cloud project using `gcloud`.

#### Prerequisites
* Have the Google Cloud CLI (`gcloud`) installed and authenticated.
* Administrator permissions (`roles/iam.workloadIdentityPoolAdmin` and `roles/iam.serviceAccountAdmin`) in the target project.

#### 1. Get the Project Number
Save your project number (not the alphanumeric ID) in an environment variable to make the following commands easier.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash 
export PROJECT_ID="your-project-id"
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
```
</div>

#### 2. Create the Workload Identity Pool
The "Pool" is the logical container that will group the external identities.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash 
gcloud iam workload-identity-pools create "github-actions-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="GitHub Actions Pool"
```
</div>

#### 3. Create the OIDC Provider
The "Provider" establishes the connection and defines how GitHub token attributes are mapped to Google Cloud.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-actions-pool" \
  --display-name="GitHub OIDC Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```
</div>

#### 4. Bind the Service Account to the Repository
This step authorizes a specific GitHub repository to generate tokens on behalf of your GCP Service Account.

!!! note
    Replace `your-organization/your-repository` with the exact values.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
export SERVICE_ACCOUNT="your-service-account@${PROJECT_ID}.iam.gserviceaccount.com"
export REPO="your-organization/your-repository"

gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/${REPO}"
```
</div>


#### 5. Get the Provider Identifier
Extract the full path of the generated provider, which you will need in your GitHub pipeline.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
gcloud iam workload-identity-pools providers describe "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-actions-pool" \
  --format="value(name)"
```
</div>

!!! success "Obtaining the provider identifier"
    (Copy the output of this command, it will look like: `projects/123456789/locations/global/workloadIdentityPools/...`)






## 2. Checkov {: #checkov }

### What is it and what is it for?

Checkov is an open-source tool for detecting security vulnerabilities in infrastructure code.
