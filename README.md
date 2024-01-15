# Notas Kubernetes CKAD 

Notes for the Kubernetes CKAD taken during the Udemy/KodeKloud exam preparation course.

## Content

Based on the approximate % of the exam:

- Application Design and Build - 20%
  - [Define, build and modify container images](ckad-02-config-05-container_images.md)
  - Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
    - [Core concepts](ckad-01-core_concepts.md)
    - [Labels & selectors](ckad-05-pod_design-01-labels_selectors.md)
    - [Jobs & CronJobs](ckad-05-pod_design-04-jobs_cronjobs.md)
  - [Understand multi-container Pod design patterns (e.g. sidecar, init and others)](ckad-03-multi_container_pods.md)
  - [Utilize persistent and ephemeral volumes](ckad-07-state_persistence_volumes.md)

- Application Deployment - 20%
  - [Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)](ckad-05-pod_design-03-deployment_strategies.md)
  - [Understand Deployments and how to perform rolling updates](ckad-05-pod_design-02-roll_deployments.md)
  - [Use the Helm package manager to deploy existing packages](ckad-05-pod_design-05-helm.md)
  - Kustomize

- Application Observability and Maintenance - 15%
  - [Understand API deprecations](ckad-04-api-deprecation.md)
  - [Implement probes and health checks](ckad-04-observability.md#1-probes)
  - [Use built-in CLI tools to monitor Kubernetes applications](ckad-04-observability.md#4-monitoring)
  - [Utilize container logs](ckad-04-observability.md#3-logs)
  - [Debugging in Kubernetes](ckad-04-observability.md#5-debug-pods)

- Application Environment, Configuration and Security - 25%
  - Discover and use resources that extend Kubernetes (CRD, Operators)
  - Understand authentication, authorization and admission control
  - [Understand requests, limits, quotas](ckad-02-config-02-resources.md)
  - [Understand ConfigMaps](ckad-02-config-01-properties.md)
  - [Define resource requirements](ckad-02-config-04-affinity.md)
  - [Create & consume Secrets](ckad-02-config-01-properties.md)
  - [Understand ServiceAccounts](ckad-02-config-03-security.md#2-service-account)
  - [Understand Application Security (SecurityContexts, Capabilities, etc.)](ckad-02-config-03-security.md#1-security-context)

- Services and Networking - 20%
  - [Demonstrate basic understanding of NetworkPolicies](ckad-06-services_network-03-networking.md)
    - [Network Policy Example](ckad-06-services_network-04-network_policy_example.md)
  - [Provide and troubleshoot access to applications via services](ckad-06-services_network-01-services.md)
  - [Use Ingress rules to expose applications](ckad-06-services_network-02-ingress.md)

### Additional information

There are other files with various exercises and notes:
- [Complete YAML manifests example](ckad-99-others-02-example_complete.md)
- [Warm up exercises for the exam](ckad-99-others-08-warm_up.md)
- Solutions to KodeKloud exercises
  - [Lightning lab 1](ckad-99-others-04-lightning_lab_1.md)
  - [Lightning lab 2](ckad-99-others-05-lightning_lab_2.md)
  - [Mock exam 1](ckad-99-others-06-mock_exam_1.md)
  - [Mock exam 2](ckad-99-others-07-mock_exam_2.md)
- [Horizontal Autoscaling - NOT INCLUDED IN CKAD](ckad-99-others-01-horizontal_autoscaling.md)
- [Notes about default requirements](ckad-note_detault_resource_req.md)
- [Notes about editing Pods](ckad-note_edit-pods.md)
- [Tips about formatting](ckad-tips_formatting.md)

### Deprecated index, previous to 2021 changes

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
  - [Jobs & cronjobs](ckad-05-pod_design-04-jobs_cronjobs.md)
- Services and networking - 13%
  - [Services](ckad-06-services_network-01-services.md)
  - [Ingress](ckad-06-services_network-02-ingress.md)
  - [Network policy](ckad-06-services_network-03-networking.md)
- [State persistence - 8%](ckad-07-state_persistence_volumes.md)