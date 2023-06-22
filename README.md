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

Anyways, it worked fine on my EKS cluster <br/>
We will start testing on a single-node EKS cluster

![Screenshot 2023-06-22 at 13 56 26](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/514a5da0-2c04-4332-b2a8-2b1385165d45)


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

Note: This image includes the Python runtime but no specific additional dependencies required by my Python script. <br/>
If my script has additional dependencies, I will either need to customize the Dockerfile or build my own image that includes those dependencies.

## Testing the Python Script

```
import subprocess
from falco import Client

# Set up the Falco client
falco_client = Client()

# Define the action to be taken when a CRITICAL Falco alert is triggered
def handle_critical_alert(alert):
    container_id = alert.output.get("container.id")
    if container_id:
        # Terminate the container by sending a SIGKILL signal
        subprocess.run(["docker", "kill", container_id])
        print(f"Container {container_id} terminated.")

# Subscribe to Falco alerts and handle them accordingly
falco_client.subscribe("output", lambda alert: handle_critical_alert(alert))

# Start the Falco client
falco_client.run()
```

![Screenshot 2023-06-22 at 13 59 57](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/c80a96fc-33a1-4619-b3b0-0dc70c0bf1a1)

I need to ensure I have the necessary dependencies installed, including the ```falco-python``` library <br/>
Seems after running the below command, there is no matching dependency called ```falco-python```

```
pip install falco-python
```

![Screenshot 2023-06-22 at 14 03 30](https://github.com/nigeldouglas-itcarlow/local-k8s-lab-security/assets/126002808/71af040b-4e4b-4e0b-83db-d8d895ab3dd2)


I will also need to have Falco installed and properly configured on my system first <br/>
<br/>
This updated script will listen for Falco alerts and trigger the ```handle_critical_alert``` function when a ```CRITICAL```-level alert is received. <br/>
It should extract the ```container.id``` field from the alert output and use it to terminate the corresponding container using the ```docker kill``` command.
