# Intro to Kubernetes: Hands-On

Welcome to this introductory Hands-On session for Kubernetes. 

### Table of Contents

* [1. Starting Minikube](#1-starting-minikube) 
* [2. Introduction to kubectl](#2-introduction-to-kubectl) 
* [3. Starting a Pod - `kubectl run`](#3-starting-a-pod---kubectl-run) 
* [4. Deploying an app - `kubectl apply`](#4-deploying-an-app---kubectl-apply) 
* [5. Stateless vs Stateful Apps](#5-stateless-vs-stateful-apps) 
* [6. Troubleshooting](#6-troubleshooting) 
* [7. Stopping Minikube](#stopping-minikube) 
* [Resources](#resources) 

### 1. Setting up the repo and minikube
------

- Clone this repository to your local machine

`git clone https://github.com/conmurphy/kubernetes-hands-on`

`cd kubernetes-hands-on`

- If you haven't already, follow the instructions from the Minikube website to install and start Minikube

[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

> Note: Remember to start Docker Desktop if using docker as driver

- Start Minikube and set the CNI flag to Calico

`minikube start --driver=docker --network-plugin=cni --cni=calico --nodes=2`

- Wait for all system pods to be in the `Running 1/1` state

`kubectl get pods -n kube-system`

### 2. Introduction to kubectl
------
You can interact with Kubernetes via CLI tool, Kubectl. Kubectl talks to the Kubernetes API. When starting Minikube, kubectl will automatically be configured to connect to the minikube cluster. 

> Note: A Kubernetes cluster is made up of a number of control plane and worker nodes.

- View the Kubernetes nodes that make up the cluster

`kubectl get nodes` 

- View more details using the `-o wide` flag 

`kubectl get nodes -o wide`

- Describe the nodes to see more details

`kubectl describe nodes`

- You can also describe an individual node (or other Kubernetes resource) using the node name found in the `kubectl get nodes` command.

`kubectl describe node <node-name>`

> Note: Namespaces are a logical way to group resources in a cluster into separate spaces

- Take a look at which namespaces exist by default in a Minikube cluster

`kubectl get namespaces`

> Note: Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. They contain one or more containers.

- View the pods in the default namespace by using the `-n <namespace name>` flag

`kubectl get pods -n default`

- If you don't specify a namespace, the `kubectl` comma will apply to the `default` namespace

`kubectl get pods`

- If you want to search all namespaces you can use the `-A` flag

`kubectl get pods -A`

- View all the Kubernetes system pods which implement the Kubernetes functionality

`kubectl get pods -n kube-system -o wide`

### 3. Starting a Pod - `kubectl run`
------

- You can start a pod using the `kubectl run` command and the format ` kubectl run NAME --image=image` where `NAME` is what you want to call your pod and `image` is the name of the image you want to run

`kubectl run firstpod --image=oguzpastirmaci/hostname`

- Confirm the pod is running

`kubectl get pods`

- To make this pod accessible from outside the cluster, you need to create a service.

`kubectl expose pod firstpod --port=8000 --type=LoadBalancer`

- Check the Kubernetes service has been created. You should see the `EXTERNAL-IP` is in `<pending>` status

`kubectl get svc`

> Note: Minikube has built-in functionality to allow you to access this traffic from the local machine. 

- Open a **new** terminal/cmd instance

`minikube tunnel`

- You might have to wait a few seconds before the tunnel is established. There will be no dynamic output. 

- You should see `Starting tunnel for service firstpod` which indicates the tunnel was established.

- Confirm that the service was created and and an `EXTERNAL-IP` was assigned. You may see `127.0.0.1` as the IP address

`kubectl get service` 

- Open [localhost:8000](http://localhost:8000) in the browser of your choice.

<br>
This simple app displays the hostname of the container you are connected to. Since you only have one right now, this won't change when refreshing the page. 

- You can now observe the resources

`kubectl get pods`

`kubectl describe pod firstpod`

`kubectl get services`

`kubectl describe service firstpod`


- Clean up the app resources

`kubectl delete service firstpod`

`kubectl delete pod firstpod`

- Confirm that the resources have been removed

`kubectl get all`

### 4. Deploying an app - `kubectl apply`
------

This section will again deploy the hostname application (with 2 replicas) but the settings will be stored in a configuration file rather than from the command line.

- Change the directory into the `demo04` folder

`cd demo04`

- Look through the files in the folder
  `hostname-namespace.yaml`
  `hostname-deployment.yaml`
  `hostname-service.yaml`

- Apply the configuration 

`kubectl apply -f hostname-namespace.yaml`

`kubectl apply -f hostname-deployment.yaml`

`kubectl apply -f hostname-service.yaml`

- You can look at individual resources or all resources together

`kubectl get all -n hostname`

- Confirm that two pods are running

`kubectl get pods -n hostname`

> Note: Your `minikube tunnel` session should still be open for this. Start a new one if it is closed, or if it is not working within a few seconds. 
>
> In a separate terminal window run `minikube tunnel`

- Take note of the address and port the app is running on. 

`kubectl get services -n hostname`

- In your browser open localhost on the port you see in the service we just created


- Refresh the page on your browser (ctrl/cmd + shift + r) to show change between pods. 

> Note: Every now and then the hostname should change to show that the LoadBalancer is directing traffic to a new Pod. Caching mechanisms might prevent the page from refreshing properly to see the hostname change. 

- If you don't see the hostname change and both hostname pods are running, try `curl http://localhost:port` in your Terminal to see the LoadBalancing mechanism switch between the pods.

`curl http://localhost:port`

- Edit the `hostname-deployment.yaml` file to have 3 replicas

- Change the number next to `replicas: ` to `3` and save the file

- Update the configuration on your cluster 

`kubectl apply -f hostname-deployment.yaml`

You should see an output: `deployment.apps/hostname configured`

- Confirm that **three** pods are now running

`kubectl get pods -n hostname`

- Cleanup the resouces

`kubectl delete -f hostname-namespace.yaml`

> Note: Deleting a namespace will delete all resources within it (it might take some time as the pods terminate)

- Confirm that the resources have been removed

`kubectl get all -n hostname`

### 5. Stateless vs Stateful Apps
------

- Change directory into `demo05`

`cd demo05`

- Look at `message-board-all-in-one.yaml`

- Apply the configuration

`kubectl apply -f message-board-all-in-one.yaml`

- Check the resources were created 

`kubectl get all -n message-board`

- Watch the resources as they're created by using the `-w` flag

`kubectl get pods -n message-board -w`

- Wait for the pod to be `running` 

- Once the pods are running, press `ctrl + c` to go back. 

> Note: Your `minikube tunnel` session should still be open for this. Start a new one if it is closed, or if it is not working within a few seconds. 
>
> In a separate terminal window run `minikube tunnel`

- Open [localhost:5000](http://localhost:5000)

- Sign up to the message board, log in, and write a message

- Refresh the page and confirm that the message still remains

- Delete the pod, confirm it was deleted and that a new one is starting

`kubectl delete pod -l name=message-board -n message-board`

`kubectl get pods -n message-board`

- Reload the page 

- Confirm that the message is still there.

> Note: This is because the message was saved in a Persistent Volume, which gets attached to the pod. 

- Cleanup the resouces

`kubectl delete -f message-board-all-in-one.yaml`

- Confirm that the resources have been removed

`kubectl get all -n message-board`

### 6. Troubleshooting

#### 6a. Following Pod Logs

- If the `minikube tunnel` from previous sections was closed then open a new one

`minikube tunnel`

- Create a new Nginx pods

`kubectl create deployment --image nginx my-nginx`

- Confirm the pod is running

`kubectl get pods`

- To make this pod accessible from outside the cluster, you need to create a service.

`kubectl expose deployment my-nginx --port=30001 --target-port=80 --type=LoadBalancer`

- Watch the Nginx logs as you access a browser 

`kubectl logs deployment/my-nginx --follow` 

- Access a browser on `http://localhost:30001` and refresh the page a couple of times. You should see the Nginx welcome page

- Go back to your terminal and confirm you can see the logs updating as you refresh the page

- Cleanup the resouces

`kubectl delete deployment my-nginx`

- Confirm that the resources have been removed

`kubectl get all`

#### 6b. Troubleshooting Image and Storage Errors

- Change directory into `demo06`

`cd demo06`

- Look at `message-board-all-in-one-troubleshooting.yaml`

- Apply the configuration

`kubectl apply -f message-board-all-in-one-troubleshooting.yaml`

- Check if the pods have started

`kubectl get pods -n message-board`

- If the pod is not in `running` status check the events for any errors. What do you find?

`kubectl describe pods -n message-board`

- Check the storage has been provisioned

`kubectl get persistentvolumeclaim -n message-board`

- If the status is `pending` check the events for errors. What do you find?

`kubectl describe pvc -n message-board`

- Cleanup the resouces

`kubectl delete -f  message-board-all-in-one-troubleshooting.yaml -n message-board`

- Confirm that the resources have been removed

`kubectl get all -n message-board`

### Stopping Minikube
------

- If you are finished with the lab you can stop the cluster

`minikube stop`

- This stops the minikube cluster, but retains the state for when you start the cluster again via `minikube start`.

- To completely delete the cluster you can run `minikube delete`

## Resources

[Official Kubernetes.io CheatSheet Page](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)




