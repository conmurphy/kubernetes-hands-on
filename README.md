# Intro to Kubernetes: Hands-On

Welcome to this introductory Hands-On session for Kubernetes. 

## 0. Starting Minikube

Reference on how to install minikube, as well as potential errors, check the [website](https://minikube.sigs.k8s.io/docs/start/).

``` minikube start``````


> :warning: Remember to start docker desktop if using docker

:warning: If you have an older installation of Minikube and are getting errors, try:
minikube delete --all --purge
docker system prune

On Mac: 
You can change kubectl to the alias k - stops typos and is faster. 
The second command enables completion
alias k=kubectl
complete -F __start_kubectl k

## 1. Introduction to kubectl
```kubectl get nodes / get no```/n
```kubectl get namespaces / get ns```/n
```kubectl get pods -A / get po -A```/n
```kubectl ```/n


## 2. Run pod
```kubectl run firstpod --image=oguzpastirmaci/hostname```
```kubectl expose pod firstpod --port=8000 --type=LoadBalancer```

```new terminal```
```minikube tunnel```

kubectl get service - check localhost and port number
open localhost:8000

kubectl get service
kubectl describe service firstpod
kubectl get pods
kubectl describe pod firstpod

kubectl delete service firstpod
kubectl delete pod firstpod

## 3. Declarative Deployment
look at hostname-namespace, deployment, and service YAML files 
kubectl apply -f hostname-namespace.yaml
kubectl apply -f hostname-deployment.yaml
kubectl apply -f hostname-service.yaml

kubectl get all -n hostname

open localhost:port in output
if not working, restart minikube tunnel

edit deployment .yaml to 2-3 replicas

kubectl apply -f hostname-deployment.yaml -n hostname

refresh page (ctrl/cmd + shift + r) to show change between pods

kubectl delete -f namespace.yaml
kubectl delete -f hostname-deployment.yaml -n hostname
kubectl delete -f hostname -service -n hostname

## 4. Stateless vs Stateful 
look at message-board-all-in-one.yaml
kubectl apply -f message-board-all-in-one.yaml
kubectl get all -n message-board

minikube tunnel
localhost:5000
access website - sign up and write something

kubectl get pods
kubectl delete pod message-board-xxxxx

reload website - data still there even though pod was deleted

## 5. Network Policy
look at guestbook-all-in-one
kubectl apply -f guestbook-all-in-one.yaml

kubectl get pods

minikube tunnel
open up website
localhost:80
write a note into the guestbook

kubectl apply -f network-policy.yaml
look at policy - no ingress traffic allowed into redis

reload website - message disappeared because no comms
might take a few moments
kubectl delete -f network-policy.yaml

reload website - message appears again
might take a few moments


https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf


