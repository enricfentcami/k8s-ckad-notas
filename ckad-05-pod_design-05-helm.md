# POD DESIGN - Using Helm

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

## **1. Using Helm**

The main command is the intall command, used to install the application/package:

`helm install wordpress`

Customize settings before install the package/app using the `values.yaml` file:

Values.yaml file sample:

```yaml
image: wordpress:5.0
worpressUsername: user
wordpressEmail: user@example.com
wordpressFirstName: FirstName
wordpressLastName: LastName
```

Using values.yaml file in manifests:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  USER_NAME: {{ .Values.worpressUsername }}
```

Install the chart using the custom values:

`helm install -f values.yaml mywordpress wordpress`

Or set values one by one (same attributes as values.yaml file):

`helm install --set image=wordpress:5.0 --set worpressUsername=user mywordpress wordpress`


Other useful commands:

* List installed packages: `helm list`
* Install chart setting a release name: `helm install release-1 wordpress`
* Upgrade: `helm upgrade wordpress`
* Rollback: `helm rollback wordpress`
* Uninstall: `helm uninstall wordpress`
* Search in Helm Chart Hub: `helm search hub wordpress`
* Search in othe repos:
  * `helm repo add bitnami https://carts.bitnami.com/bitnami`
  * `helm search repo wordpress`
* List repos: `helm repo list`
* Download and extract package: `helm pull --untar wordpress`
  * Check `wordpress` folder that contains the config files including the values.yaml
  * Install the files: `helm install release-1 ./wordpress`
