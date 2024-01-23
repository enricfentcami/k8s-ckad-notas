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

* current-context: Set the context used by kubectl by default
* clusters: Define the clusters configuration
  * Name of the cluster
  * Server URL
  * Certificates: Configure the full path of the certificate, field `certificate-authority` or use the certificate data in base 64 format, field `certificate-authority-data`.
* contexts: Define the context config, mapping cluster and users
* users: Define the users of the clusters and his credentials

#### Useful commands

View config: `kubectl config view`

View custom file: `kubectl config view --kubeconfig=my-custom_config`

Update current context: `kubectl config use-context prod-user@production`

Set default namespace: `kubectl config set-context --current --namespace=prod`

## **2. Authorization**

## **3. Admission Control**
