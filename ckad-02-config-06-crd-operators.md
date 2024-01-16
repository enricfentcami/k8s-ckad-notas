# CONFIGURATION - CRD and Operators

Docs: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/

## **1. Concepts**

### Custom resources

A resource is an endpoint in the Kubernetes API that stores a collection of API objects of a certain kind; for example, the built-in pods resource contains a collection of Pod objects.

A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. It represents a customization of a particular Kubernetes installation. However, many core Kubernetes functions are now built using custom resources, making Kubernetes more modular.

### Custom controllers

On their own, custom resources let you store and retrieve structured data. When you combine a custom resource with a custom controller, custom resources provide a true declarative API.

The Kubernetes declarative API enforces a separation of responsibilities. You declare the desired state of your resource. The Kubernetes controller keeps the current state of Kubernetes objects in sync with your declared desired state. This is in contrast to an imperative API, where you instruct a server what to do.

### CustomResourceDefinitions

The CustomResourceDefinition API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource. The name of a CRD object must be a valid DNS subdomain name.

## **2. Custom Resource Definitions**

Custom Resource example:
```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

CRD example:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: flighttickets.flights.com
spec:
  # either Namespaced or Cluster
  scope: Namespaced
  # group name to use for REST API: /apis/<group>/<version>
  group: flights.com
  names:
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: FlightTicket
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: flightTickets
    # singular name to be used as an alias on the CLI and for display
    singular: flightTicket
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ft
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  ninimum: 1
                  maximum: 10
```

### Commands

Get CRDs in the cluster: `kubectl get crd`

Get details of a CRD: `kubectl describe crd globals.traffic.controller`

### Examples

#### Example 1

Create a CRD and a custom resource.

CRD example definition:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com 
spec:
  group: datasets.kodekloud.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

Custom Resource for the datasets CRD:
```yaml
kind: Internal
apiVersion: datasets.kodekloud.com/v1
metadata:
  name: internal-space
  namespace: default
spec:
  internalLoad: "high"
  range: 80
  percentage: "50"
```

#### Example 2

Create a new resource that defines an object of "traffic.controller/v1"

Search the CRD definition and check the Group, Kind, Scope and Properties

`kubectl get crd`

```console
NAME                               CREATED AT
collectors.monitoring.controller   2024-01-16T12:55:20Z
globals.traffic.controller         2024-01-16T12:55:21Z
internals.datasets.kodekloud.com   2024-01-16T12:58:37Z
```

`kubectl describe crd globals.traffic.controller`

```console
Name:         globals.traffic.controller
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2024-01-16T12:55:21Z
  Generation:          1
  Resource Version:    501
  UID:                 c231b0f5-77ab-41ad-880a-ba41ade84bde
Spec:
  Conversion:
    Strategy:  None
  Group:       traffic.controller
  Names:
    Kind:       Global
    List Kind:  GlobalList
    Plural:     globals
    Short Names:
      gb
    Singular:  global
  Scope:       Namespaced
  Versions:
    Name:  v1
    Schema:
      openAPIV3Schema:
        Properties:
          Spec:
            Properties:
              Access:
                Type:  boolean
              Data Field:
                Type:  integer
            Type:      object
        Type:          object
    Served:            true
    Storage:           true
Status:
  Accepted Names:
    Kind:       Global
    List Kind:  GlobalList
    Plural:     globals
    Short Names:
      gb
    Singular:  global
  Conditions:
    Last Transition Time:  2024-01-16T12:55:21Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2024-01-16T12:55:21Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1
Events:  <none>
```

Create the resource:
```yaml
apiVersion: traffic.controller/v1
kind: Global
metadata:
  name: datacenter
spec:
  dataField: 2
  access: true
```

## **3. Operator Framework**

Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components.

Used to deploy Custom Resource Definions and Custom Controllers of a component. Example: Operator to deploy a RabbitMQ cluster definition.

OperatorHub.io is the main repository of Operators. 