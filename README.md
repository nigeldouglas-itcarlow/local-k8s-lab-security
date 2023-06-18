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
