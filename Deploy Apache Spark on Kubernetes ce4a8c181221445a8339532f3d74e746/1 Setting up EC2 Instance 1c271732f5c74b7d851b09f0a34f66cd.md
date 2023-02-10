# 1. Setting up EC2 Instance

Assuming that you have learned how to set up and have **********************************Launch an EC2 instance …**********************************

Start up your AWS Instance

![Untitled](1%20Setting%20up%20EC2%20Instance%201c271732f5c74b7d851b09f0a34f66cd/Untitled.png)

Once the Instance State is ***Ready*** click Connect, to connect to your AWS Session Manager in your EC2 Instance.

![Untitled](1%20Setting%20up%20EC2%20Instance%201c271732f5c74b7d851b09f0a34f66cd/Untitled%201.png)

In order to run Kubernetes cluster on AWS EC2 instance, install minikube.

```jsx
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Once installed, check you minikube version 

```jsx
minikube version
```

Start your minikube

```jsx
minikube start --vm-driver=none
```

Check minikube status to check if minikube is running on your ec2 instance

```jsx
minikube status
```