**Set up enviorenment**
- Setup Virtual machine in Virtual Box
- If virtual support manager not enabled , then run these  commands on local machine ( vm must be stopped then) : 
 
           Vboxmanage modifyvm “vm_name”  --nested-hw-virt=on 

- Now install  Minicube for Kubernetes Cluster. For this , 
Open terminal to this location and run : 
            
           Sudo chmod +x minikube.sh
          
           ./minikube.sh

**How to run?**


minikube start 

minikube addons enable ingress

kubectl apply -f namespace.yaml

kubectl apply -f deployments.yaml

kubectl apply -f services.yaml

kubectl apply -f ingress.yaml

kubectl get ingress -n sample (this may take some time to display address. this address equals minikube ip)

