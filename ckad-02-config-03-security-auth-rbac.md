# CONFIGURATION - Security - Authentication, Authorization and Admission Control

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

_Note: This not a recommended authentication mechanism_

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

Used to authoriza to non namespaced resources like: nodes, PVs, namespaces, ...
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

## **3. Admission Control**
