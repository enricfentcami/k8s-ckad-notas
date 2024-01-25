# POD DESIGN - Jobs & CronJobs

## **1. Jobs**

https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

By default, Kubernetes when running a Pod that executes an action and terminates, executes it again and again due to the `restartPolicy` policy which defaults to `Always`. To make it run once you would do it with the `Never` policy.

Pod example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
    - name: math-pod
      image: ubuntu
      command: ['expr','3','+','2']
    restartPolicy: Never
```

### **1.1. Define a Job**

A manager is needed that can create as many Pods as are required to perform a task to complete, for example in parallel data processing.

Job example:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

### **1.2. Commands**

`kubectl create -f job-definition.yaml`

`kubectl get jobs`

You can see the Pod created with the STATUS `completed` with 0 RESTARTS

`kubectl get pods`

In this case the result is in the Pod log:

`kubectl logs math-add-job-xxxxx`

> `5`

Delete the job:

`kubectl delete job math-add-job`

#### Examples in real life

Image processing, reports + email sending, migration process...

### **1.3. Multiple Pods**

The number of SUCCESSFUL executions you want to have will be defined by the attribute `completions`. Pods are created sequentially when the previous one has finished.

Job example:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

When a job fails, another is executed until the number of SUCCESSFUL executions that had been defined with `completions` is completed.

### **1.4. Multiple Pods in parallel**

The number of Pods to be created in parallel is indicated with `parallelism`.

Job example:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

It will create 3 Pods at a time and if any fail it will create them again until those indicated in `completions` are completed.

### **1.5. Limit the number of attempts**

The maximum number of failures allowed is defined with the `backoffLimit` attribute.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  backoffLimit: 6
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

## **2. CronJobs**

Jobs scheduled as 'crontab'.

CronJob example:
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *" # Cron expression
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: reporting-tool
              image: reporting-tool
            restartPolicy: Never
```

### **2.2. Commands**

`kubectl create -f cron-job-definition.yaml`

`kubectl get cronjob`

Delete the cronJob:

`kubectl delete cronjob reporting-cron-job`

Create a job from an existing cronjob:

`kubectl create job --from=cronjob/pgdump pgdump-manual-001`
