# MInikube Cluster : 
###  VM Setup

1. Setup Virtual machine in Virtual Box
If virtual support manager not enabled , then run these commands on local machine ( vm must be stopped then) : 
 
 ```sh
 Vboxmanage modifyvm “vm_name”  --nested-hw-virt=on
 ```

2. Now install  Minicube for Kubernetes Cluster. 

### Minikube.sh : 
```sh
#!/bin/bash

sudo apt-get update -y
sudo apt-get install curl wget apt-transport-https virtualbox virtualbox-ext-pack -y
echo "1st install docker"
sudo apt update && apt -y install docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo chmod 666 /var/run/docker.sock

echo "Apply updates"
sudo apt update -y
sudo apt upgrade -y

echo " Download Minikube Binary"
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo cp minikube-linux-amd64 /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube
minikube version


echo "Install Kubectl utility"
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version -o yaml

echo "Start the minikube"
minikube start
minikube status
```


Open terminal to this location and run : 
            
    Sudo chmod +x minikube.sh
          
    ./minikube.sh


Start Cluster :
 ```sh
 minicube start
 ```
### Deploy a demo application 

**Deploy application (service):**

Create a sample deployment and expose it on port 8080:
```sh 
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```
It may take a moment, but your deployment will soon show up when you run:
```sh
kubectl get services hello-minikube
```
 The easiest way to access this service is to let minikube launch a web browser for you:
```sh
minikube service hello-minikube
```
Alternatively, use **kubectl** to forward the port:
```sh
kubectl port-forward service/hello-minikube 7080:8080
```
Tada! Your application is now available at http://localhost:7080/.

Source : 
https://minikube.sigs.k8s.io/docs/

#### Deployment of Fastapi-Application

**Docker Container:** 
<Fastapi-hello-world files >
**_Dockerfile  |  main.py  |  requirements.txt_**

 Build the docker image :
```sh
docker build -t galib9940/fastapi-hello-world .
```
Docker Image Name: galib9940/fastapi-hello-world
Docker Hub Username: galib9940
Repository Name: fastapi-hello-world
Tag: latest (default)

Check images : 
```sh
docker images
```
Run the docker Container : 
```sh
docker run -d -p 8000:80 --name fastapi-container galib9940/fastapi-hello-world
```
Run a container from your existing Docker image. This will map port 8000 on your host to port 80 in the container.
Check the container : 
```
docker ps 
docker logs fastapi-container
```
Access the FastAPI application:  curl http://localhost:8000


Docker Compose : 
For multiple container application 
Need a docker-compose.yaml file


#### Deployment Hierarchy: 
##### _Ingressing <- Service <- Deployments <- Replica Set <- Pods_

- Pods >> container , because multiple pods can be running at the same time.

- Service is served like a proxy for multiple pods ip. 
If client hit on the service port , then service forwards to the appropriate port.

- memorizing ip address is difficult , so hitting the application with a domain name. 
That’s why ingressing concept is needed!  

#### Ingressing
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.

![alt text](https://earthly.dev/blog/assets/images/mutual-tls-kubernetes-nginx-ingress-controller/ptpr1xB.png )

Enabling ingress 
```
Minikube addons enable ingress 
```
Yaml file creating (services,pods,ingress) 
```
Kubectl apply -f <file.yaml>
```
If namespace is created , then… ( isolate resources : deployment, services, ingress)
```
Kubectl get pods -n <namespace_name>
Kubectl get service -n <namespace_name>
Kubectl get ingress -n <namespace_name>
```
#### Domain Resolution
Ensuring domain name (galib123.com) points to the correct ip address of your ingress controller or minikube cluster 

1. Verify ip:( ip will be same for minikube cluster and ingress controller )
```
minikube ip  
Kubectl get ingress -n <namespace> o -wide
```
 
2. DNS configuration : nslookup galib123.com (should display the ip address with your domain)

If failed , then -> 
Make sure that , Minikube ip or ingress controller ip is reachable from your host machine!! 

To check : ping ip_address ( sends ICMP echo req and wait for response)
3. Check ingress controller(nginx) is running properly or not. 
```
kubectl get pods -n kube-system -l app.kubernetes.io/name=ingress-nginx
```
This command filters the pods in the kube-system. If not found , then create a one . 
4. To check all from ingress-nginx:
```
kubectl get all -n ingress-nginx
```
#### Nginx-Ingress-Controller
**Still getting error while hitting  at the domain galib123.com** ??
So the mechanism is basically , when i hit at domain , it forwards to ingress controller external ip ( which is basically virtual machine’s host only adapter ip :  192.168.56.101  & this is the ip should be  config in local host (/etc/hosts)

Cli command : 
```
cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	bs949
192.168.56.101    galib123.com (defined)
```
But unfortunately , we can’t deine external ip into the minikube cluster Nginx controller service. So a routing table should be created , so that ,
**_Domain hit -> External ip -> Routing table -> ingress controller ip -> application_**

But this is not a good practice. So, you can use Load Balancer as a service. 
otherwise, use kubeadm cluster instead of minikube.

link: https://docs.google.com/document/d/1Q770dZRTpCCU8lMtP1kYFogKq8sUH6Q15IaNDttCFqI/edit?pli=1
