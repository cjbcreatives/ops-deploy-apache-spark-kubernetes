# 2. Set up Splunk Connect for Kubernetes

Terms:

- helm
    - is a package manager for Kubernetes that makes it easy to take applications and services that are either highly repeatable or used in multiple scenarios and deploy them to a typical K8s cluster.
- FQDN
    - Fully Qualified Domain Name
    - is the complete domain name for a specific computer, or host, on the internet. The FQDN consists of two parts: the hostname and the domain name.
- HEC Token
    - HTTP Event Collector
    - a fast and efficient way to send data to Splunk Enterprise and Splunk Cloud Platform.

Install helm to your EC2 Instance

```python
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Add Splunk chart repo

```jsx
helm repo add splunk https://splunk.github.io/splunk-connect-for-kubernetes/
```

Get values file in your working directory

```jsx
helm show values splunk/splunk-connect-for-kubernetes > values.yaml
```

Configure values.yaml by modifying the following global variables:

- host : <FQDN>
- token <HEC token>
- protocol: http
- indexName: sandbox
- clusterName: minikube

```jsx
global:
  splunk:
    hec:
      host: < FQDN of Splunk Enterprise Instance - Will be provided by mentor >
      token: < "HEC token from setup or obtain this from Splunk Web under Data Inputs > HTTP Event Collector" >
      protocol: http
      indexName: sandbox
    kubernetes:
      clusterName: "minikube"
```

After preparing the Values file, you can simply install the chart with by running:

```jsx
helm install my-splunk-connect -f values.yaml splunk/splunk-connect-for-kubernetes
```

To update configuration

```jsx
helm upgrade my-splunk-connect -f values.yaml splunk/splunk-connect-for-kubernetes
```

Note: **Do not uninstall**, this is just to show how to uninstall Splunk Connect

```jsx
helm uninstall my-splunk-connect
```

Create a new namespace your kubernetes cluster

```jsx
kubectl create namespace <namespace-name>
```

Check if the namespace is created

```python
kubectl get pods -n <namespace-name>
# result would be: no pods is created in <nsname> namespace

# Another way to check 
kubectl get ns | grep <namespace-name>
```

Create your Splunk configuration yaml file.

```jsx
vi values.yaml
```

Save and run the Splunk config file once again

```jsx
helm upgrade my-splunk-connect -f values.yaml splunk/splunk-connect-for-kubernetes
```

Now that we have our Splunk configured on our EC2 instance…

Let’s try to check if our EC2 instance is connected to our Splunk Web…

Open the given Splunk link, log in

Go to Search & Reporting

On the search bar use the following query to check if your EC2 instance is connected

```jsx
//Splunk
//check splunk to search namespace using query
index=sandbox | stats count by namespace
```

![Untitled](2%20Set%20up%20Splunk%20Connect%20for%20Kubernetes%209566a4745eba49b0a45e4021721ba853/Untitled.png)