# 4. Creating a Spark Application using a Docker image.

### Build a docker image

Create Python Script

```jsx
vi pi-sleep300.py
```

```python
import sys
from random import random
import time
from operator import add

from pyspark.sql import SparkSession

if __name__ == "__main__":
    """
        Usage: pi [partitions]
    """
    spark = SparkSession\
        .builder\
        .appName("PythonPiSleep")\
        .getOrCreate()

    partitions = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    n = 100000 * partitions

    def f(_):
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 <= 1 else 0

    count = spark.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    time.sleep(300)
    print("Pi is roughly %f" % (4.0 * count / n))

    spark.stop()
```

Add executable permission for the new python file

```python
chmod +x pi-sleep300.py
```

Create a Dockerfile

```python
vi Dockerfile
```

```python
FROM gcr.io/spark-operator/spark-py:v3.1.1-hadoop3
LABEL maintainer="NAME HERE <emailhere@opswerks.com>"

USER root
RUN groupadd -g 185 spark && \
    useradd -u 185 -g 185 spark

RUN chown -R spark:spark /opt/spark/
RUN chown -R spark:spark /tmp/

USER spark

RUN mkdir /opt/spark/logs
RUN mkdir /tmp/spark-events

ADD pi-sleep300.py /opt/spark/examples/src/main/python
```

Build a docker image based from Dockerfile

```python
docker build -t 431522258999.dkr.ecr.us-west-2.amazonaws.com/<name>-academy-batch-4-app:<tag> .
```

Log in to your ECR

```python
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 431522258999.dkr.ecr.us-west-2.amazonaws.com 
```

Push docker image to your ECR

```python
docker push 431522258999.dkr.ecr.us-west-2.amazonaws.com/<name>-academy-batch-4-app:<tag>
```

### Create a SparkApplication

Create a new namespace

```python
kubectl create namespace <namespace-name>
```

### RBAC authorization

- Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

Create a RBAC authorization file to your namespace

```python
vim rbac.yaml
```

```python
# Service Account Creation
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    meta.helm.sh/release-name: werks-lab
    meta.helm.sh/release-namespace: spark-system
  labels:
    app.kubernetes.io/instance: werks-lab
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

Create regcred secrets for docker-registry

```python
kubectl -n <namespace-name> create secret docker-registry regcred \
  --docker-server=431522258999.dkr.ecr.us-west-2.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(/usr/bin/aws ecr get-login-password --region us-west-2)
```

Create the SparkApplication yaml file

```python
vim <spark-app-name>.yaml
```

```python
apiVersion: "sparkoperator.k8s.io/v1beta2"
kind: SparkApplication
metadata:
  name: spark-job-pi
spec:
  type: Python
  pythonVersion: "3"
  mode: cluster
  image: "431522258999.dkr.ecr.us-west-2.amazonaws.com/<name>-academy-batch-4-app:<tag>"
  imagePullPolicy: Always
  mainApplicationFile: "local:///opt/spark/examples/src/main/python/pi-sleep300.py"
  sparkVersion: "3.1.1"
  sparkConf:
    "spark.kubernetes.container.image.pullSecrets": "regcred"
  timeToLiveSeconds: 3
  restartPolicy:
    type: OnFailure
    onFailureRetries: 3
    onFailureRetryInterval: 10
    onSubmissionFailureRetries: 5
    onSubmissionFailureRetryInterval: 20
  driver:
    cores: 1
    coreLimit: "1500m"
    memory: "1024m"
    labels:
      version: 3.1.1
      driver-pod: spark-py-pi
    serviceAccount: spark-job-sa
  executor:
    cores: 1
    instances: 1
    memory: "512m"
    labels:
      version: 3.1.1
      executor-pod: spark-py-pi
```

```python
kubectl create -f <spark-app-name>.yaml -n <namespace-name>
```

### Access the SparkApplication Web UI

Create a service yaml file

```python
vim <service-name>.yaml
```

```python
apiVersion: v1
kind: Service
metadata:
  name: spark-py-pi-driver-svc
spec:
  type: NodePort
  selector:
    driver-pod: spark-py-pi
  ports:
    - name: spark-py-pi-driver-port
      port: 4040
      targetPort: 4040
      nodePort: <nodeport> #between 30000-32000 ports
```

```python
kubectl create -f <service-name>.yaml -n <namespace-name>
```

To access the SparkApplication UI using the browser, follow the format below. Make sure this port is allowed in the instance’s security group.

```python
<aws public IPv4 DNS>:<nodePort>
```

If the driver pod terminates after 300 seconds, 

```python
kubectl apply -f <spark-app-name>.yaml -n <namespace-name>
```