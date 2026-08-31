# GitOps & Infrastructure Provisioning (Reducing Toil)

## IDP - Internal Developer Platform

### Backstage (Developer Portal)
An open-source platform developed by Spotify that acts as a "developer portal" or "Steel Toe Shoes for developers". Its main goal is to provide a centralized place where software teams can find, navigate, and manage all their software products, services, documentation, tools, and more.
[https://backstage.io/](https://backstage.io/)


### Score (Open Core) - Developer Abstraction (Workload Specification)
Open-source platform for IT service lifecycle management.
[https://score.dev/](https://score.dev/)

### Crossplane - Infrastructure API (Infra as Code) 

The Engine and Control Plane (Orchestration and Infrastructure)
Open-source platform for IT service lifecycle management.
[https://www.crossplane.io/](https://www.crossplane.io/)

## KOps
The easiest way to get a production-grade Kubernetes cluster up and running.

[https://kops.sigs.k8s.io/](https://kops.sigs.k8s.io/)

## Kubespray
Deploy a production-ready Kubernetes cluster on-premise.

[https://kubespray.io/#/](https://kubespray.io/#/)


## Kubernetes Cluster API (CAPI)
Cluster API is a Kubernetes sub-project focused on providing declarative APIs and tooling to simplify provisioning, upgrading, and operating multiple Kubernetes clusters.

Started by the Kubernetes Cluster Lifecycle Special Interest Group (SIG), the Cluster API project uses Kubernetes-style APIs and patterns to automate cluster lifecycle management for platform operators. The supporting infrastructure, like virtual machines, networks, load balancers, and VPCs, as well as the Kubernetes cluster configuration, are defined in the same way that application developers deploy and manage their workloads. This enables consistent and repeatable cluster deployments across a wide variety of infrastructure environments.

[https://cluster-api.sigs.k8s.io/](https://cluster-api.sigs.k8s.io/)


## Use Cases

1. **Security Hardening:** Automatic application of security patches (CIS Benchmarks).
2. **Disaster Recovery:** Scripts for automated database restoration.

## GitOps Deployment Patterns

To manage clusters at scale and handle multiple applications or environments, GitOps utilizes declarative patterns. Two of the most important in the Argo CD ecosystem are the **App of Apps** pattern and **ApplicationSets**.

### 1. "App of Apps" Pattern

The **App of Apps** pattern is a technique where a single "Root App" is defined, which, instead of deploying direct resources (like Pods or Services), deploys `Application` type manifests (other Argo CD applications).

#### Key Benefits
*   **Easy Bootstrapping:** Allows you to stand up an entire cluster (core, data, apps) by applying a single root resource.
*   **Hierarchical Structure:** Ideal for maintaining an organized repository, grouping applications by infrastructure layers.
*   **Centralized Management:** A single entry point to apply version control and auditing to the entire topology.

### 2. ApplicationSets (The Evolution)

While "App of Apps" requires manually creating an `Application` manifest for each child app or cluster, **ApplicationSet** is a native Argo CD controller designed to automate and generate applications dynamically based on *Generators*.

#### App of Apps vs. ApplicationSet

| Feature | App of Apps | ApplicationSet |
| :--- | :--- | :--- |
| **App Generation** | Manual (requires writing an `Application` manifest for each app/cluster). | Automatic (based on Git, Cluster, List, Matrix generators, etc.). |
| **Primary Use** | Static bootstrapping of a single cluster or fixed base infrastructure. | Dynamic multi-cluster management (e.g., deploying the app to all clusters labeled as `prod`). |
| **Scalability** | Limited. Multiple clusters require copy/pasting manifests. | High. A single `ApplicationSet` can deploy hundreds of apps across dozens of clusters. |
| **Maintenance** | Can become verbose and difficult to maintain as it grows. | Centralized templates, reducing boilerplate code and making global updates easier. |

**Conclusion:** "App of Apps" is excellent for getting started and structuring static logical dependencies. However, for dynamic, multi-tenant, or multi-cluster deployments, **ApplicationSet** is the modern standard and the natural evolution of the pattern.
