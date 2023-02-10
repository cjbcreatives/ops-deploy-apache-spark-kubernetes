# 3. Kubernetes on Spark Operator

![Untitled](3%20Kubernetes%20on%20Spark%20Operator%20563ebebf5c044a47b48f1f77bcc17d3f/Untitled.png)

## Spark Operator

- According to Google, Spark Operator is **a Kubernetes custom controller that uses custom resources for declarative specification of Spark applications;** it also supports automatic restart and cron-based, scheduled applications.

### Create the spark operator

Install the operator, using helm.

```jsx
$ helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator
```

Install the spark operator

```jsx
helm install my-release spark-operator/spark-operator --namespace <namespace-operator> --create-namespace
```

Grant cluster-admin privileges to your spark operator before you can create custom roles and role bindings

```jsx
kubectl create clusterrolebinding ssm-user-cluster-admin-binding --clusterrole=cluster-admin --user=ssm-user@[ec2-35-90-17-135.us-west-2.compute.amazonaws.com](http://ec2-35-90-17-135.us-west-2.compute.amazonaws.com/)
```

Now you should see the operator running in the cluster by checking the status of the Helm release.

****

```jsx
helm status --namespace <namespace-operator> my-release
```

### Add the 3 Components: role, role binding and service account to your spark exporter

Create a service account 

```jsx
kubectl create serviceaccount spark --namespace=<namespace-operator>
```

Create cluster-rolebinding

```jsx
kubectl create clusterrolebinding spark-operator-role --clusterrole=cluster-admin --serviceaccount=<namespace where s.a. is>:<s.account name → spark> --namespace=<namespace-operator>
```

Check if service account is attached to the cluster role

```jsx
helm install my-release spark-operator/spark-operator --namespace spark-operator --set sparkJobNamespace=<namespace-operator>
```

Note: Don’t run jobs under the spark operator

![Untitled](3%20Kubernetes%20on%20Spark%20Operator%20563ebebf5c044a47b48f1f77bcc17d3f/Untitled%201.png)

### Create new namespace and the 3 components to the namespace, to run a job.

```jsx
kubectl create serviceaccount spark --namespace=<namespace-job>

kubectl create clusterrolebinding spark-operator-role --clusterrole=cluster-admin --serviceaccount=default:<s.account name → spark> --namespace=<namespace-job>

vi spark-pi.yaml
// edit the namespace: <namespace-job>

kubectl apply -f spark-pi.yaml -n spark-test-caryl
// update the file
```

### Another way to create the service account, role, and rolebinding via RBAC yaml file.

*NOTE: before editing the rbac.yaml ….* 

*Stop Instance and change the instance type to >> t2.xlarge*

Create the rbac.yaml

```jsx
touch rbac.yaml
vi rbac.yaml
```

Edit rbac.yaml

```jsx
# Service Account Creation
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    meta.helm.sh/release-name: werks-lab
    meta.helm.sh/release-namespace: spark-system
  labels:
    app.kubernetes.io/instance: werks-lab
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: spark-operator
    app.kubernetes.io/version: v1beta2-1.3.8-3.1.1
    helm.sh/chart: spark-operator-1.1.26
  name: spark-job-sa

---

# Role Creation
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    meta.helm.sh/release-name: werks-lab
    meta.helm.sh/release-namespace: spark-system
  labels:
    app.kubernetes.io/instance: werks-lab
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: spark-operator
    app.kubernetes.io/version: v1beta2-1.3.8-3.1.1
    helm.sh/chart: spark-operator-1.1.26
  name: spark-job-role
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - persistentvolumeclaims
  verbs:
  - '*'

---

# Rolebinding Creation
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  annotations:
    meta.helm.sh/release-name: werks-lab
    meta.helm.sh/release-namespace: spark-system
  labels:
    app.kubernetes.io/instance: werks-lab
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: spark-operator
    app.kubernetes.io/version: v1beta2-1.3.8-3.1.1
    helm.sh/chart: spark-operator-1.1.26
  name: spark-job-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: spark-job-role
subjects:
- kind: ServiceAccount
  name: spark-job-sa
```

Edit the spark-pi.yaml —> service accounts name = service accounts >> name from rbac.yaml

Apply the changes

```jsx
kubectl apply -f rbac.yaml -n spark-test-caryl
kubectl apply -f spark-pi.yaml -n spark-test-caryl
```

Run sparkapplication to check

![Untitled](3%20Kubernetes%20on%20Spark%20Operator%20563ebebf5c044a47b48f1f77bcc17d3f/Untitled%202.png)

```jsx
kubectl get sparkapplication spark-pi -n spark-test-caryl
// check if job is "COMPLETED"
```

### Testing in creating a job in a new namespace.

![Untitled](3%20Kubernetes%20on%20Spark%20Operator%20563ebebf5c044a47b48f1f77bcc17d3f/Untitled%203.png)

Now you have a spark operator and successfully created a job.