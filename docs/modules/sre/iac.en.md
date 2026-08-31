# Infrastructure as Code (IaC)

This section centralizes :simple-terraform: [Terraform](https://www.terraform.io/) templates designed for the creation of secure infrastructure resources in [Google Cloud](https://cloud.google.com/), as well as On-Prem using :simple-ansible: [Ansible](https://docs.ansible.com/), also github actions and github packages for workflow orchestration.

## 1. Cloud Infrastructure Implementation
##### Code Access
The complete source code is located in the `iac_gke` repository, and we have opted to structure the repository in a modular way with Terragrunt to allow reusability across different environments (Dev, Staging, Prod).

[Source Code on GitHub :octicons-link-external-16:](https://github.com/mcatalangt/iac_gke.git){ .md-button  }


##### Technical Specifications 
|Type|Provider|Nodes| Node Type |Specs| RAM |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Dev GKE | GCP |2| e2-medium | 2 vCPU| 4 GB |
| Stage GKE | GCP |2| n1-standard-2 | 2 vCPU| 7.5 GB |
| Prod GKE | GCP |3| e2-standard-2 | 2 vCPU| 8 GB |

## 2. On-Prem Infrastructure Deployment
### Kubespray 
It is an open-source tool (belonging to the official Kubernetes SIGs project) designed to automate the deployment, configuration, and maintenance of production-ready Kubernetes clusters.

## 3. Stack (The Ingredients)
!!! note "Tools"
    * **Cloud:** Google Cloud Platform (GCP) [GKE, Compute Engine]
    * **Tool:** Terraform v1.5+, Terragrunt, Ansible, GitHub Actions, Github Package
    * **Security:** IAM Least Privilege, VPC Service Controls
## 4. Architecture
The code is modularized to allow reusability in different environments (Dev, Staging, Prod) in Cloud and On-Prem.

##### Repository Structure:
I am using the DRY (Don't Repeat Yourself) design pattern of Terragrunt, which separates the global configuration from the specific configuration of each module.

<div style="font-size: 0.60rem; line-height: 1.2;">

```text
📦 iac_core    --- Base Repository
 ┣ 📂 .github
 ┃ ┗ 📂 workflows
 ┃   ┣ 📜 deploy.yaml  --- Infrastructure deployment workflow
 ┃   ┗ 📜 destroy.yaml --- Infrastructure destruction workflow
 ┣ 📂 live
 ┃ ┣ 📂 desarrollo
 ┃ ┃ ┣ 📂 gke-base
 ┃ ┃ ┃ ┗ 📜 terragrunt.hcl    ---  Terragrunt configuration for the base cluster
 ┃ ┃ ┗ 📂 gke-resources
 ┃ ┃   ┗ 📜 terragrunt.hcl    ---  Terragrunt configuration for internal resources (namespaces, etc)
 ┃ ┗ 📜 terragrunt.hcl        ---  Global ROOT terragrunt configuration (State and Providers)
 ┣ 📂 modules
 ┃ ┗ 📂 gke-base
 ┃   ┗ 📜 main.tf           ---  Terraform module for infrastructure deployment
 ┃   ┗ 📜 outputs.tf        --- Module outputs
 ┃   ┗ 📜 variables.tf        --- Module variables
 ┃ ┗ 📂 gke-resources
 ┃   ┗ 📜 main.tf           --- Terraform module for resource deployment
 ┃   ┗ 📜 outputs.tf        --- Module outputs
 ┃   ┗ 📜 variables.tf        --- Module variables
 ┗ 📜 README.md             --- Repository README
```

</div>

##### Architecture Diagram:
![Architecture](../../assets/BasicGKE.png){ align=center width="100%" }


## 5. Step by Step

##### 1. Download the code from the repository

```bash 
git clone https://github.com/mcatalangt/iac_gke.git 
```

##### 2. Create 2 environment variables in GitHub
- `GCP_PROJECT`: Set the project ID in GCP (e.g. platform-core-386722)
- `GCP_REGION`: Set the region or zone name in GCP (e.g. us-central1)
    
![variables](../../assets/variablesGitHub.png){ align=center width="100%" }

##### 3. Create a Workload Identity Federation in GCP
We will use this to authenticate github actions with GCP without using keys.

👉 [View configuration guide](security.md#workload-identity)

##### 4. GitHub Actions Implementation

!!! warning "Important:"
    The `permissions` block is mandatory for GitHub to generate the `OIDC token`, and `export_environment_variables: true` is crucial for tools like `Terraform/Terragrunt` to detect the temporary token in subsequent steps.

Modify the following block in your `.github/workflows/deploy.yml` file.

<div style="font-size: 0.60rem; line-height: 1.2;">

```bash
jobs:
  deploy:
    runs-on: ubuntu-latest
    
    # Mandatory requirement to request the OIDC token
    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
      - name: 'Checkout Code'
        uses: 'actions/checkout@v4'

      - name: 'Authenticate with Workload Identity'
        id: 'auth'
        uses: 'google-github-actions/auth@v2'
        with:
          workload_identity_provider: 'projects/YOUR_PROJECT_NUMBER/locations/global/workloadIdentityPools/github-actions-pool/providers/github-provider'
          service_account: 'your-service-account@your-project-id.iam.gserviceaccount.com'
          export_environment_variables: true 

      - name: 'Execute deployment (E.g. Terraform/Terragrunt)'
        run: 'terragrunt apply -auto-approve'
```
</div>

##### 5. Infrastructure Deployment
You can run the code from GitHub Actions or from your local machine using git push


## 6. E2E Validation

##### Execute Deploy in GitHub Actions
![Deploy Actions](../../assets/githubActions.png){ align=center width="100%" }


##### GKE Overview
![GKE](../../assets/gke.png){ align=center width="100%" }


##### GKE Resources Overview
![GKE](../../assets/gke-resources.png){ align=center width="100%" }

##### Execute Destroy in GitHub Actions
![Destroy Actions](../../assets/destroyActions.png){ align=center width="100%" }

## 7. Other Included Modules

| Module| Description |Status| Repository |
| :--- | :--- | :--- | :--- |
| `01-iac-postgresql` | HA PostgreSQL DB creation. | ✅ Stable | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/01-iac-postgresql) |
| `02-iac-mysql` | HA MySQL DB creation. | ✅ Stable | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/01-iac-postgresql) |
| `03-iac-mongodb` | HA MongoDB DB creation. | ✅ Stable | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/01-iac-postgresql) |
| `04-iac-neo4j` | HA Neo4J DB creation. | ✅ Stable | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/01-iac-postgresql) |
| `05-iac-prefect` | Prefect Workflow creation, orchestrator and workflow automator| 🚧 Beta | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/02-iac-prefect) |
| `06-iac-event-driven` | Event driven creation (PubSub, Kafka, RabbitMQ) for message management and system decoupling| 🚧 Beta | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/03-iac-event-driven) |
| `07-iac-kubernetes` | Kubernetes creation on GKE, Container Orchestrator in 5 minutes| ✅ Stable | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/04-iac-kubernetes) |
| `08-iac-observability` | Grafana Stack creation on GKE for E2E transactional system Observability (Logs, Traces, Metrics and Profiles)| 🚧 Beta | [GitHub :octicons-link-external-16:](https://github.com/mcatalangt/data-reliability-hub/tree/main/05-iac-observability) |
