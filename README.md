# Notas Kubernetes CKAD 

Notas para la CKAD de Kubernetes tomadas durante el curso de Udemy/KodeKloud de preparación del examen.

## Contenido

Basado en el % aproximado de que consta el examen:

- [Core Concepts - 13%](ckad-01-core_concepts.md)
- Configuration - 18%
  - [ConfigMap & Secrets](ckad-02-config-01-properties.md)
  - [Resources](ckad-02-config-02-resources.md)
  - [Security context & Service account](ckad-02-config-03-security.md)
  - [Node affinity](ckad-02-config-04-affinity.md)
- [Multi-container pods - 10%](ckad-03-multi_container_pods.md)
- [Observability - 18%](ckad-04-observability.md)
- Pod design - 20%
  - [Labels & selectors](ckad-05-pod_design-01-labels_selectors.md)
  - [Deployments: Rolling updates and rollbacks](ckad-05-pod_design-02-roll_deployments.md)
  - [Jobs & cronjobs](ckad-05-pod_design-03-jobs_cronjobs.md)
- Services and networking - 13%
  - [Services](ckad-06-services_network-01-services.md)
  - [Ingress](ckad-06-services_network-02-ingress.md)
  - [Network policy](ckad-06-services_network-03-networking.md)
- [State persistence - 8%](ckad-07-state_persistence_volumes.md)



Application Design and Build - 20%
Define, build and modify container images
Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
Understand multi-container Pod design patterns (e.g. sidecar, init and others)
Utilize persistent and ephemeral volumes

Application Deployment - 20%
Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
Understand Deployments and how to perform rolling updates
Use the Helm package manager to deploy existing packages
Kustomize

Application Observability and Maintenance - 15%
Understand API deprecations
Implement probes and health checks
Use built-in CLI tools to monitor Kubernetes applications
Utilize container logs
Debugging in Kubernetes

Application Environment, Configuration and Security - 25%
Discover and use resources that extend Kubernetes (CRD, Operators)
Understand authentication, authorization and admission control
Understand requests, limits, quotas
Understand ConfigMaps
Define resource requirements
Create & consume Secrets
Understand ServiceAccounts
Understand Application Security (SecurityContexts, Capabilities, etc.)

Services and Networking - 20%
Demonstrate basic understanding of NetworkPolicies
Provide and troubleshoot access to applications via services
Use Ingress rules to expose applications


Hay otros ficheros con ejercicios variados y notas.