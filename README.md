# Intro to Kubernetes: Hands-On

Welcome to this introductory Hands-On session for Kubernetes. 

### Table of Contents

[1. Starting Minikube](#1-starting-minikube)

[Resources](#resources)


[Introduction to kubectl](https://github.com/cedricfeist/kubernetes-hands-on#1-introduction-to-kubectl)

[Starting a Pod - Imperative](https://github.com/cedricfeist/kubernetes-hands-on#2-starting-a-pod---imperative)

[Declarative Deployments](https://github.com/cedricfeist/kubernetes-hands-on#3-declarative-deployments)

[Stateless vs Stateful Apps](https://github.com/cedricfeist/kubernetes-hands-on#4-stateless-vs-stateful-apps)

[Network Policy](https://github.com/cedricfeist/kubernetes-hands-on#5-network-policy)


### 1. Setting up the repo and minikube
------

- Clone this repository to your local machine

`git clone https://github.com/conmurphy/kubernetes-hands-on`

`cd kubernetes-hands-on`

- If you haven't already, follow the instructions from the Minikube website to install and start Minikube

[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

> Remember to start Docker Desktop if using docker as driver

- Start Minikube and set the CNI flag to Calico

`minikube start --driver=docker --network-plugin=cni --cni=calico --nodes=2`

- Wait for all system pods to be in the `Running 1/1` state

`kubectl get pods -n kube-system`

### 2. Introduction to kubectl
------
You can interact with Kubernetes via CLI tool, Kubectl. Kubectl talks to the Kubernetes API. When starting Minikube, kubectl will automatically be configured to connect to the minikube cluster. 

> A Kubernetes cluster is made up of a number of control plane and worker nodes.

- View the Kubernetes nodes that make up the cluster

`kubectl get nodes` 

- View more details using the `-o wide` flag 

`kubectl get nodes -o wide`

- Describe the nodes to see more details

`kubectl describe nodes`

- You can also describe an individual node (or other Kubernetes resource) using the node name found in the `kubectl get nodes` command.

`kubectl describe node <node-name>`

> Namespaces are a logical way to group resources in a cluster into separate spaces

- Take a look at which namespaces exist by default in a Minikube cluster

`kubectl get namespaces`

> Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. They contain one or more containers.

- View the pods in the default namespace by using the `-n <namespace name>` flag

`kubectl get pods -n default`

- If you don't specify a namespace, the `kubectl` comma will apply to the `default` namespace

`kubectl get pods`

- If you want to search all namespaces you can use the `-A` flag

`kubectl get pods -A`

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

<br>
Minikube has built-in functionality to allow you to access this traffic from the local machine. 
<br>
<br>

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

### 4. Deploying an app - `kubectl apply`
------

This section will again deploy the hostname application (with 2 replicas) but the settings will be stored in a configuration file rather than from the command line.

- Change the directory into the `demo04` folder

`cd demo04`

- Look through the files in the folder
  - `hostname-namespace.yaml`
  - `hostname-deployment.yaml`
  - `hostname-service.yaml`

- Apply the configuration 

`kubectl apply -f hostname-namespace.yaml`

`kubectl apply -f hostname-deployment.yaml`

`kubectl apply -f hostname-service.yaml`

- You can look at individual resources or all resources together

`kubectl get all -n hostname`

- Confirm that two pods are running

`kubectl get pods -n hostname`

> Your `minikube tunnel` session should still be open for this. Start a new one if it is closed, or if it is not working within a few seconds. 

- Take note of the address and port the app is running on. 

`kubectl get services -n hostname`

- In your browser open localhost on the port you see in the service we just created


- Refresh the page on your browser (ctrl/cmd + shift + r) to show change between pods. 

> Every now and then the hostname should change, to show that the LoadBalancer is directing traffic to a new Pod. Caching mechanisms might prevent the page from refreshing properly to see the hostname change. 

> Try `curl http://localhost:port` in your Terminal to see the LadBalancing mechanism switch between the pods. 


- Edit the hostname-deployment.yaml file to have 3 replicas

- Change the number next to `replicas: ` to `3` and save the file

- Update the configuration on your cluster 

`kubectl apply -f hostname-deployment.yaml`

You should see an output: `deployment.apps/hostname configured`

- Cleanup the resouces

`kubectl delete -f hostname-namespace.yaml`


Deleting a namespace will delete all resources within it. 

### 5. Stateless vs Stateful Apps
------

- Change directory into `demo05`

`cd demo05`

- Look at `message-board-all-in-one.yaml`

> Note that instead of keeping each resource in a separate file, all have been combined into a single file.

- Apply the configuration

`kubectl apply -f message-board-all-in-one.yaml`

- Check the resources were created 

`kubectl get all -n message-board`

- Watch the resources as they're created by using the `-w` flag

`kubectl get pods -n message-board -w`

- Wait for The Pod to be Running and Ready. 

- Once the pods are running, press `ctrl + c` to go back. 

- If the `minikube tunnel` from previous sections was closed then open a new one

- `minikube tunnel`

- Open [localhost:5000](http://localhost:5000)

- Sign up to the message board, log in, and write a message

- Refresh the page and confirm that the message still remains

- Delete the pod, confirm it was deleted and that a new one is starting

`kubectl delete pod -l name=message-board -n message-board`

`kubectl get pods -n message-board`

- Reload the page 

- Confirm that the message is still there.

> This is because the message was saved in a Persistent Volume, which gets attached to the pod. 



### 5. Network Policy
------
The last thing we will look at in this session is Network Policies. 

Before applying the configuration file, look at *guestbook-all-in-one.yaml* file

```
kubectl apply -f guestbook-all-in-one.yaml
```
```
kubectl get pods -w
```

> Wait for all the Pods to be Running and Ready

:warning: Once the pods are Ready you can press `ctrl + c` to go back. 

> If not running yet enter `minikube tunnel` in a second window

Run `kubectl get service` to verify the port:

> Open [localhost:80](http://localhost:80) on your browser

:warning: Port 80 will require root privileges, so check your `minikube start` window to enter your password, if the page doesn't open. 

> Write a note into the guestbook

More detailed information on Kubernetes Network Policies can be found [here](https://kubernetes.io/docs/concepts/services-networking/network-policies/) but for now we will create a simple one. 

> Take a look at the network policy in *network-policy.yaml*. All it does is prevent ingress traffic to the redis services to stop the frontend from retrieving messages.

```
kubectl apply -f network-policy.yaml
```

> Reload the webpage. Within a few moments the new policy should be enforced and the message disappear

> :warning: This can take a few seconds. 

> :warning: Caching mechanisms might prevent the page from refreshing properly to see the message disappear. 
> Try `curl http://localhost:port` in your Terminal to load the current state of the page, and see notice the missing messages in the message <div>. 

```
kubectl delete -f network-policy.yaml
```

> Reload the webpage again

It might take a few more moments as before, but eventually the message will reappear as the policy allows traffic to the redis pods once again. 


### Stopping Minikube
------

- If you are finished with the lab you can stop the cluster

`minikube stop`

- This stops the minikube cluster, but retains the state for when you start the cluster again via `minikube start`.

- To completely delete the cluster you can run `minikube delete`

## Resources

[Official Kubernetes.io CheatSheet Page](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)




