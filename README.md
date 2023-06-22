# local-k8s-lab-security
Setting-up a local Kubernetes lab for security  <br/>

![screenshot](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/1f5d3859-7c39-4f25-b464-252b88fce8aa)

If you already have ```kubectl``` installed, you can now use it to access your shiny new cluster:
```
kubectl get pods -A
```
Alternatively, ```minikube``` can download the appropriate version of kubectl and you should be able to use it like this:
```
minikube kubectl -- get pods -A
```
You can also make your life easier by adding the following to your shell config:
```
alias kubectl="minikube kubectl --"
```
## Remove unwanted Interfaces

I was experiencing multi-interface issues: <br/>
https://github.com/kubernetes/minikube/issues/13131
```
hostonlyif remove "VirtualBox Host-Only Ethernet Adapter"
```
After removing the interface, checked that it was removed before proceeding:
```
VBoxManage list hostonlyifs
```

![remove](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/079b4db4-bdb5-48d7-a1eb-26f334b0dca9)

## Installing helm via chocolatey

![helm](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/2ff6b904-9b35-44ff-8c0d-94335a638b54)

## Install Falco
```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco --namespace falco --create-namespace
```

```
kubectl get pods -n falco -o wide -w
```

Falco pod was slow initializing:
![error](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/1ba52ff4-19cc-4c41-a25d-d8dbaf76bb91)


Get the logs from the pod falco-rq4gs
```
kubectl logs falco-rq4gs -n falco
```

Mon Jan 30 10:56:26 2023: Falco version: 0.33.1 (x86_64)
Mon Jan 30 10:56:26 2023: Falco initialized with configuration file: /etc/falco/falco.yaml
Mon Jan 30 10:56:26 2023: Loading rules from file /etc/falco/falco_rules.yaml
Mon Jan 30 10:56:26 2023: Loading rules from file /etc/falco/falco_rules.local.yaml
Mon Jan 30 10:56:27 2023: The chosen syscall buffer dimension is: 8388608 bytes (8 MBs)
Mon Jan 30 10:56:27 2023: Starting health webserver with threadiness 4, listening on port 8765
Mon Jan 30 10:56:27 2023: Enabled event sources: syscall
Mon Jan 30 10:56:27 2023: Opening capture with Kernel module

## Building a process killer workload

There are already a variety of container images available that include Python and commonly used dependencies. <br/>
One such example is the official Python Docker image, which provides a base image with Python pre-installed.  <br/>
I could use different tags of the Python image to specify the Python version you need. I choice 3.9 as it is newer.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: falco-alert-handler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: falco-alert-handler
  template:
    metadata:
      labels:
        app: falco-alert-handler
    spec:
      containers:
      - name: falco-alert-handler
        image: python:3.9
        command: ["python", "script.py"]
```
