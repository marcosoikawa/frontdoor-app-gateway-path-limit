# Bypass limit of path of Front Door / Application Gateway
This sample objective to handle the path limit of 200 origins of Front Door and / or 100 backends of Application Gateway, for very large deployments / migrations that all applications is under a unique URL and distributed by paths

## Architecture diagram

## Prerequisites
- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

---

## Configure networking

Set up the virtual networking for the environment

```bash
az group create \
    --name fd-appg-pahtlimit \
    --location brazilsouth
```

#### Create Cluster 01

```bash
# Create AKS 01
az aks create --resource-group fd-appg-pahtlimit --name aks01 --enable-app-routing --enable-managed-identity --node-count 1 --generate-ssh-keys

#credentials
az aks get-credentials -n aks01 -g fd-appg-pahtlimit

#namespace
kubectl create namespace aks-app

#Deployment
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/aks01.yaml -n aks-app

#service
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/service.yaml -n aks-app

#ingress
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/ingress01.yaml -n aks-app

#verify
kubectl get ingress -n aks-app

```
#### Create Cluster 02

```bash
# Create AKS 02
az aks create --resource-group fd-appg-pahtlimit --name aks02 --enable-app-routing --enable-managed-identity --node-count 1 --generate-ssh-keys

#credentials
az aks get-credentials -n aks02 -g fd-appg-pahtlimit

#namespace
kubectl create namespace aks-app

#Deployment
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/aks02.yaml -n aks-app

#service
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/service.yaml -n aks-app

#ingress
kubectl apply -f https://raw.githubusercontent.com/marcosoikawa/frontdoor-app-gateway-path-limit/refs/heads/main/deployments/ingress02.yaml -n aks-app

#verify
kubectl get ingress -n aks-app

```

#### Create Application Gateway

```bash

#create vnet
az network vnet create --name pathlimit-vnet --resource-group fd-appg-pahtlimit --location brazilsouth --address-prefix 10.22.0.0/16 --subnet-name appgtwsubnet --subnet-prefix 10.22.0.0/24

#create public ip
az network public-ip create --resource-group fd-appg-pahtlimit --name pathlimit-pip --allocation-method Static --sku Standard

#create Application Gateway
az network application-gateway create --name pathlimit-appgtw --location brazilsouth --resource-group fd-appg-pahtlimit --capacity 2 --sku Standard_v2 --public-ip-address pathlimit-pip --vnet-name pathlimit-vnet --subnet appgtwsubnet --priority 100

```
```bash

#get AKSs Ingress IPs

#AKS01
az aks get-credentials -n aks01 -g fd-appg-pahtlimit
kubectl get ingress

#AKS01
az aks get-credentials -n aks02 -g fd-appg-pahtlimit
kubectl get ingress

```