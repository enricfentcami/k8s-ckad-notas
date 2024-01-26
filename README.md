# Notes Kubernetes CKAD 

Notes for the Kubernetes CKAD taken during the Udemy/KodeKloud exam preparation course: https://www.udemy.com/course/certified-kubernetes-application-developer

Reference documentation:
* Kubectl quick reference: https://kubernetes.io/docs/reference/kubectl/quick-reference/ 
    (_It is recommended to have it open during the exam as an entry point to the official documentation_)
* Use the kubectl `explain` command to get all resource fields in the CLI:
  * `kubectl explain pod`
  * `kubectl explain pod.spec`
  * `kubectl explain pod.spec.containers.envFrom`

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
  - Kustomize: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

- Application Observability and Maintenance - 15%
  - [Understand API deprecations](ckad-04-api-deprecation.md)
  - [Implement probes and health checks](ckad-04-observability.md#1-probes)
  - [Use built-in CLI tools to monitor Kubernetes applications](ckad-04-observability.md#4-monitoring)
  - [Utilize container logs](ckad-04-observability.md#3-logs)
  - [Debugging in Kubernetes](ckad-04-observability.md#5-debug-pods)

- Application Environment, Configuration and Security - 25%
  - [Discover and use resources that extend Kubernetes (CRD, Operators)](ckad-02-config-06-crd-operators.md)
  - [Understand authentication, authorization and admission control](ckad-02-config-03-security-auth-rbac.md)
  - [Understand requests, limits, quotas](ckad-02-config-02-resources.md)
  - [Understand ConfigMaps](ckad-02-config-01-properties.md)
  - [Define resource requirements](ckad-02-config-04-affinity.md)
  - [Create & consume Secrets](ckad-02-config-01-properties.md)
  - [Understand ServiceAccounts](ckad-02-config-03-security-apps.md#2-service-account)
  - [Understand Application Security (SecurityContexts, Capabilities, etc.)](ckad-02-config-03-security-apps.md#1-security-context)

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
- [Notes about editing Pods](ckad-note_edit-pods.md)
- [Tips about formatting](ckad-tips_formatting.md)
