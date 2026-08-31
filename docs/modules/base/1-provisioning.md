## Resumen Ejecutivo

Este módulo detalla el diseño arquitectónico y el aprovisionamiento automatizado de una infraestructura base resiliente de nube híbrida (Capa Core). Utilizando paradigmas avanzados de Infraestructura como Código (IaC) con Terragrunt y Kubespray, esta capa establece clústeres de Kubernetes de alta disponibilidad tanto en Google Kubernetes Engine (GKE) como en entornos on-premise. El objetivo estratégico es imponer una paridad estricta entre entornos, eliminar el "configuration drift" mediante principios DRY (Don't Repeat Yourself) robustos, e incorporar una postura de seguridad Zero-Trust a través de una autenticación OIDC perfecta (Workload Identity Federation). Esto establece una base escalable, segura y rentable para una Plataforma Interna de Desarrolladores (IDP) de nivel empresarial.

## Visión General de la Arquitectura

### La Arquitectura del "Seed Cluster"
El Seed Cluster (Clúster Semilla) es la capa de gestión fundacional dentro de una jerarquía multi-clúster.

Actuando como un entorno de alojamiento, el Seed Cluster proporciona los recursos necesarios de cómputo, almacenamiento y red para ejecutar los planos de control (control planes) de múltiples clústeres de carga de trabajo dependientes (conocidos como clústeres Shoot).

Cada plano de control de un clúster Shoot se ejecuta en su propio espacio de nombres (namespace) estrictamente aislado dentro del Seed Cluster, garantizando la independencia operativa y evitando interferencias entre clústeres.

Al desacoplar las herramientas de gestión de las cargas de trabajo de negocio, el Seed Cluster asegura que tus pipelines de entrega continua y la orquestación de infraestructura permanezcan altamente disponibles, incluso si un nodo de trabajo secundario experimenta una falla catastrófica.

## ADRs - Registros de Decisiones de Arquitectura

## Estrategia IaC

## Otros Proyectos
### 1. Cluster API (CAPI)
Cluster API (CAPI) es un subproyecto de código abierto de Kubernetes que proporciona una API nativa de Kubernetes y declarativa para la gestión del ciclo de vida de los clústeres en diversos entornos.

Utilizando el mismo patrón de controlador que Kubernetes utiliza internamente, CAPI abstrae las complejidades del aprovisionamiento de infraestructura, redes y gestión del plano de control.

Al definir las configuraciones de los clústeres objetivo en manifiestos YAML, los equipos de plataforma pueden automatizar la creación, el escalado y la actualización fluida de los clústeres como recursos estándar de Kubernetes.

Este enfoque estandariza las operaciones multi-nube, asegura una estricta compatibilidad con GitOps y automatiza de forma segura la conciliación continua entre el estado deseado y el actual de la infraestructura.

### 2. Proyecto Gardener (Kubeception)
Project Gardener es un sistema de código abierto diseñado para aprovisionar y gestionar flotas masivas de clústeres de Kubernetes de manera consistente y escalable.

Opera bajo el principio de "Kubeception", que aprovecha Kubernetes para gestionar otros clústeres de Kubernetes.

En lugar de depender de máquinas virtuales externas para los nodos maestros, Gardener despliega los componentes del plano de control (API server, controller manager, etcd) de los clústeres de usuarios finales como pods estándar en contenedores dentro de un clúster de gestión centralizado.

Este diseño reduce la sobrecarga de cómputo a través del uso compartido eficiente de recursos y permite a los equipos de plataforma escalar a miles de clústeres a través de varios proveedores de nube a gran escala (hyperscalers) o infraestructura on-premise.