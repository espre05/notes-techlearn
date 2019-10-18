

multipass shell kubevm
## Install docker
sudo apt-get install docker.io -y 
sudo usermod -aG docker ${USER}

## Configure to public docker at dockerhub
sudo docker login e6/*****

## Test using local docker registry

## Develop a golang app in OSX and push to micro-kube without uploading to dockerhub





