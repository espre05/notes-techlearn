# Laptop kubernates with Multipass & Ubuntu 

Ref: https://ubuntu.com/blog/kubernetes-on-mac-how-to-set-up

## Createvm
```
set VM_NAME=kubevm
```
#multipass delete %VM_NAME% && multipass purge && multipass launch --name %VM_NAME% --mem 4G --disk 40G
```
multipass launch --name %VM_NAME% --mem 4G --disk 40G
```


## Install micro-kube inside vm 
multipass exec %VM_NAME% -- sudo snap install microk8s --classic


## Check status but wait for ready
multipass exec kubevm -- /snap/bin/microk8s.status --wait-ready
multipass exec %VM_NAME% -- /snap/bin/microk8s.status

## Open the port     
multipass exec %VM_NAME% -- sudo iptables -P FORWARD ACCEPT 


## Check if ready
multipass list
#Name                    State             IPv4             Image
#kubevm                  Running           192.168.64.4     Ubuntu 18.04 LTS



## Connect to vm shell to update
multipass shell kubevm
#upgrade
sudo apt-get update -y
sudo apt-get upgrade -y

sudo usermod -a -G microk8s multipass
exit
#relogin so the new user is created
multipass shell kubevm

## Check status without shell
#the below config must go into file ~/.kube/config
multipass exec kubevm -- /snap/bin/microk8s.config   > multipass_kubeconfig.yaml
kubectl config view

kubectl get services
#NAMESPACE   NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
#default     service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   13m


## Enable kubernates dashboard
multipass exec kubevm -- /snap/bin/microk8s.enable dns dashboard
multipass exec kubevm -- /snap/bin/microk8s.kubectl get all --all-namespaces 

## Check status
multipass exec kubevm -- /snap/bin/microk8s.kubectl cluster-info

#Kubernetes master is running at https://127.0.0.1:16443
#Heapster is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/heapster/proxy
#CoreDNS is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
#Grafana is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
#InfluxDB is running at https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-influxdb:http/proxy


# This did not work
multipass exec kubevm -- /snap/bin/microk8s.kubectl proxy --address='0.0.0.0' --accept-hosts='.*'

# to explose local 8001 (blocking call, you can either go to aother shell, or stop this cmd to execute other )
kubectl proxy   
Grafana at:  http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy/profile

#To deploy kubernates dashbaord UI
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
Ref How to Deploy dashboard ui: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
K8 dashboard at:  http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/node?namespace=_all


multipass exec kubevm -- cat /var/snap/microk8s/current/credentials/basic_auth.csv


multipass exec kubevm -- watch /snap/bin/microk8s.kubectl get all --all-namespaces  

%%%% install kubernates dashboard, and get bearer token ######


Bearer token find: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Access dashoard locally after "kubectl proxy” : http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/node?namespace=_all

