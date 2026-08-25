# GitOps & Infrastructure Provisioning (Reducing Toil)

## IDP - Internal Developer Platform

### Backstage (Portal para Desarrolladores)
Es una plataforma de código abierto desarrollada por Spotify que actúa como un "portal para desarrolladores" o "Steel Toe Shoes para desarrolladores". Su objetivo principal es proporcionar un lugar centralizado donde los equipos de software puedan encontrar, navegar y gestionar todos sus productos de software, servicios, documentación, herramientas y más.
[https://backstage.io/](https://backstage.io/)


### Score (Open Core) - Abstracción del Desarrollador (Workload Specification)
Plataforma de código abierto para la gestión del ciclo de vida de los servicios de TI.
[https://score.dev/](https://score.dev/)

### Crossplane - API de Infraestructura (Infra as Code) 

El Motor y Control Plane (Orquestación e Infraestructura)
Plataforma de código abierto para la gestión del ciclo de vida de los servicios de TI.
[https://www.crossplane.io/](https://www.crossplane.io/)

## KOps
La forma más sencilla de poner en marcha un clúster de Kubernetes de nivel de producción.

[https://kops.sigs.k8s.io/](https://kops.sigs.k8s.io/)

## Kubespray
Implemente un clúster de Kubernetes listo para producción on-premise.

[https://kubespray.io/#/](https://kubespray.io/#/)


## Kubernetes Cluster API (CAPI)
Cluster API es un subproyecto de Kubernetes centrado en proporcionar API declarativas y herramientas para simplificar el aprovisionamiento, la actualización y el funcionamiento de múltiples clústeres de Kubernetes.

Iniciado por el Grupo de Interés Especial (SIG) de Ciclo de Vida del Clúster de Kubernetes , el proyecto Cluster API utiliza API y patrones al estilo de Kubernetes para automatizar la gestión del ciclo de vida del clúster para los operadores de la plataforma. La infraestructura de soporte, como máquinas virtuales, redes, balanceadores de carga y VPC, así como la configuración del clúster de Kubernetes, se definen de la misma manera que los desarrolladores de aplicaciones implementan y gestionan sus cargas de trabajo. Esto permite implementaciones de clúster consistentes y repetibles en una amplia variedad de entornos de infraestructura.

[https://cluster-api.sigs.k8s.io/](https://cluster-api.sigs.k8s.io/)


## Casos de Uso

1. **Security Hardening:** Aplicación automática de parches de seguridad (CIS Benchmarks).
2. **Disaster Recovery:** Scripts para la restauración automatizada de bases de datos.

## Patrones de Despliegue en GitOps

Para gestionar clústeres a escala y manejar múltiples aplicaciones o ambientes, GitOps utiliza patrones declarativos. Dos de los más importantes en el ecosistema de Argo CD son el patrón **App of Apps** y los **ApplicationSets**.

### 1. Patrón "App of Apps"

El patrón **App of Apps** es una técnica donde se define una única "Aplicación Raíz" (Root App) que, en lugar de desplegar recursos directos (como Pods o Services), despliega manifiestos de tipo `Application` (otras aplicaciones de Argo CD).

#### Beneficios Principales
*   **Bootstrapping Sencillo:** Permite levantar un clúster completo (core, data, apps) aplicando un solo recurso raíz.
*   **Estructura Jerárquica:** Ideal para mantener un repositorio ordenado, agrupando aplicaciones por capas de infraestructura.
*   **Gestión Centralizada:** Un único punto de entrada para aplicar control de versiones y auditoría a la topología completa.

### 2. ApplicationSets (La Evolución)

Mientras que "App of Apps" requiere crear manualmente un manifiesto `Application` por cada aplicación hija o clúster, **ApplicationSet** es un controlador nativo de Argo CD diseñado para automatizar y generar aplicaciones dinámicamente basándose en *generadores* (Generators).

#### App of Apps vs. ApplicationSet

| Característica | App of Apps | ApplicationSet |
| :--- | :--- | :--- |
| **Generación de Apps** | Manual (requiere escribir un manifiesto `Application` por cada app/clúster). | Automática (basada en generadores de Git, Clústeres, Listas, Matrices, etc.). |
| **Uso Principal** | Bootstrapping estático de un solo clúster o infraestructura base fija. | Gestión dinámica de multi-clúster (ej. desplegar la app en todos los clústeres etiquetados como `prod`). |
| **Escalabilidad** | Limitada. Múltiples clústeres requieren copiar/pegar manifiestos. | Alta. Un solo `ApplicationSet` puede desplegar cientos de apps en decenas de clústeres. |
| **Mantenimiento** | Puede volverse verboso y difícil de mantener a medida que crece. | Plantillas centralizadas (`template`), lo que reduce el código repetido y facilita las actualizaciones globales. |

**Conclusión:** "App of Apps" es excelente para comenzar y estructurar dependencias lógicas estáticas. Sin embargo, para despliegues dinámicos, multi-tenant o multi-clúster, **ApplicationSet** es el estándar moderno y la evolución natural del patrón.
