# CONFIGURATION - Security - Authentication, Authorization and Admission Control

kubectl -> Authentication -> Authorization -> Admission Controller -> Execute command

## **1. Authentication**

Authentication for users access to the k8s cluster for administrative purposes.

Authenticate the kube-apiserver, auth mechanisms:
* Static Password File
* Static Token File
* Certificates
* Identity Services

### Basic

* Add users and passwords in a CSV file
  * Static Password File: `password,user,id,group`
  * Static Token File: `token,user,id,group`
* Apply this auth using a command in the kube-apiserver.yaml configuration
  * Static Password File: `--basic-auth-file=user-details.csv`
  * Static Token File: `--token-auth-file=user-token-details.csv`
* Use it in the API calls specifying the basic auth. In curl `-u "user:pass"`

_Note: This is not a recommended authentication mechanism_

### KubeConfig

The access to the cluster is defined in the Kubernetes config file, used by kubectl, with the following attributes:
* current-context: Set the context used by kubectl by default
* clusters: Define the clusters configuration
  * Name of the cluster
  * Server URL
  * Certificates: Configure the full path of the certificate, field `certificate-authority` or use the certificate data in base 64 format, field `certificate-authority-data`.
* contexts: Define the context config, mapping cluster and users
* users: Define the users of the clusters and his credentials

KubeConfig file example:
```yaml
apiVersion: v1
kind: Config
current-context: research
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: development
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
users:
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
```

#### Useful commands

View config: `kubectl config view`

View custom file: `kubectl config view --kubeconfig=my-custom_config`

Update current context: `kubectl config use-context prod-user@production`

Set default namespace: `kubectl config set-context --current --namespace=prod`

## **2. Authorization**

### Role Based Access Controls

Authorization using roles for users using a set of rules. Its namespaced:
* apiGroups: For `core` group, leave it blank. For any other group we should specify the group name.
* resources: Resources to give access to: pods, deployments, services, ...
* verbs: Actions that user can take: list, get, create, update, delete, ...
* resourceNames (optional): Resource names that user can work with (In the example below: a pod with name blue or orange)

Role example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list","get","create","update","delete"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create"]
    resourceNames: ["blue", "orange"]
```

Link a user to that role using the RoleBinding:
* subject: Specify the user details
* roleRef: Provide the role created

RoleBinding example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
  namespace: default
subjects:
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

#### Check access

Check if the connected user can work with objects using the `auth can-i` command:
* `kubectl auth can-i create deployments`
* `kubectl auth can-i delete nodes`

Check using the admin user if another user can work with objects:
* `kubectl auth can-i create deployments --as dev-user`
* `kubectl auth can-i create pod --as dev-user --namespace test`

#### Useful commands

View roles: `kubectl get roles`

View role bindings: `kubectl get rolebindings`

Describe objects: `kubectl describe rolebindings devuser-developer-binding`

Check authorization modes configured on the cluster: `kubectl describe pod kube-apiserver-controlplane -n kube-system`
  Look for `--authorization-mode`

### Cluster Roles

These are Roles non namespaced, they are cluster scoped.

Used to authorize to non namespaced resources like: nodes, PVs, namespaces, ...
* Check API resources using `kubectl api-resources --namespaced=true/false`

ClusterRole example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
  - apiGroups: [""]
    resources: ["nodes","storageclasses", "persistentvolumes"]
    verbs: ["list","get","create","delete"]
```

ClusterRoleBinding example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

## **3. Admission Controller**

An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized.

Admission controllers may be validating, mutating, or both. Mutating controllers may modify objects related to the requests they admit; validating controllers may not.

Admission controllers limit requests to create, delete, modify objects. Admission controllers can also block custom verbs, such as a request connect to a Pod via an API server proxy. Admission controllers do not (and cannot) block requests to read (get, watch or list) objects.

It has a lot of pre-built admission controllers, for example:
* AlwaysPullImages
* DefaultStorageClass
* EventRateLimit
* NamespaceAutoProvision (Disabled by default, deprecated and replaced by NamespaceLifecycle)
* NamespaceExists (Enabled by default, deprecated and replaced by NamespaceLifecycle)
* NamespaceLifecycle: The NamespaceLifecycle admission controller will make sure that requests to a non-existent namespace is rejected and that the default namespaces such as default, kube-system and kube-public cannot be deleted.

View enabled admission controllers:
* `kube-apiserver -h | grep enable-admission-plugins`
* `kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins`

Enable admission controller:
* Edit the kube-apiserver object: `vi /etc/kubernetes/manifests/kube-apiserver.yaml`
* Look for `--enable-admission-plugins`

Disable admission controller:
* Edit the kube-apiserver object: `vi /etc/kubernetes/manifests/kube-apiserver.yaml`
* Look for or add `--disable-admission-plugins`

### Validating and mutating admission controller

Two types of admission controller:
* Mutating: Can change the request
* Validating: Validate the request and allow or deny

Mutating is executed before validating. For example: NamespaceAutoProvision (mutating) is executed before NamespaceExists (Validating).

#### Custom admission controller

To develop our own Admission Controllers we will use the MutatingAdmissionWebhook and ValidatingAdmissionWebhook:
1. Deploy an admission webhook server (inside the cluster or not) that receives an `AdmissionReview` object in JSON format
2. Webhook server response is another `AdmissionReview` object with the `allowed` flag to true or false
3. Webhook server can have a `validate` and/or `mutate` POST endpoints

To configure an admission webhook we need to define a YAML object:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata: 
  name: "pod-policy-example.com"
webhooks:
  - name: "pod-policy-example.com"
    clientConfig:
      # If the webhook is deployed outside the cluster
      url: "https://external-server.example.com"
      # If the webhook is deployed inside the cluster
      service:
        namespace: "webhook-namespace"
        name: "webhook-service"
      caBundle: "xxxxxx"
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
        scope: "Namespaced"
```