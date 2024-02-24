# This is a simple guide for the cheapest AKS cluster possible on Azure

## Deploy Cheap AKS Cluster

https://georgepaw.medium.com/how-to-run-the-cheapest-kubernetes-cluster-at-1-per-day-tutorial-9673f062b903

```bash
# Install az cli: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli

# Login with cli: https://www.thegeekdiary.com/az-login-command-examples-log-in-to-azure/
az login

# In case you have multifactor login for az cli tool
az login --scope https://management.core.windows.net//.default

# Fill env variables
export SUBSCRIPTION=<GUID> # az account ls
export LOCATION=switzerlandnorth # az account list-locations -o table
export RESOURCE_GROUP=<cheap-aks>
export AKS_CLUSTER=<cheap-aks>
export VM_SIZE=Standard_B2ts_v2 # az vm list-skus --location $LOCATION -o table

# Create resource group
az group create --name $RESOURCE_GROUP \
		--subscription $SUBSCRIPTION \
		--location $LOCATION

# Create a basic single-node AKS cluster
az aks create \
	--subscription $SUBSCRIPTION \
	--resource-group $RESOURCE_GROUP  \
	--name $AKS_CLUSTER \
	--node-count 1 \
	--generate-ssh-keys \
	--load-balancer-sku basic \
	--enable-cluster-autoscaler \
	--min-count 1 \
	--max-count 1 \
    --node-vm-size $VM_SIZE \
    --nodepool-name default \
    --node-osdisk-size 32 \
    --node-osdisk-type managed

# Get credentials of AKS cluster
az aks get-credentials \
	--subscription $SUBSCRIPTION \
	--resource-group $RESOURCE_GROUP \
	--name $AKS_CLUSTER \
    --admin
```

## Configure AKS Cluster

### Add Bitnami Repo for helm chart

```bash
# Add Bitnami Repo for helm
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Install NGINX Ingress

```bash
# Install nginx-ingress
helm upgrade --install nginx-ingress bitnami/nginx-ingress-controller --namespace nginx-ingress --create-namespace
```

### Install Cert-Manager for ACME

```bash
# Install cert-manager
helm upgrade --install cert-manager bitnami/cert-manager --set installCRDs=true --create-namespace -n cert-manager
# Apply Let's encrypt cluster issuer
# ⚠️ IMPORTANT: First update the email field in the letsencrypt-prod.yml file
kubectl apply -f cert-manager
```

### Install example application

```bash
# ⚠️ IMPORTANT: Update the Domain in the ingress.yml file
kubectl apply -f nginx-example
```
