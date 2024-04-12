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
export SUBSCRIPTION=<GUID> # az account list --output table -> subscriptionId
export LOCATION=switzerlandnorth # az account list-locations -o table
export RESOURCE_GROUP=<cheap-aks>
export AKS_CLUSTER=<cheap-aks>
export VM_SIZE=Standard_B2ls_v2 # az vm list-skus --location $LOCATION -o table

# Set the subcription that has to be used
az account set --subscription $SUBSCRIPTION

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

# Remove the existing node pool
az aks nodepool delete \
  --resource-group $RESOURCE_GROUP \
  --cluster-name $AKS_CLUSTER \
  --name default \
  --no-wait

# Add new node pool
az aks nodepool add \
  --resource-group $RESOURCE_GROUP \
  --cluster-name $AKS_CLUSTER \
  --name default \
  --node-count 1 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 1 \
  --node-vm-size $VM_SIZE

# Update load balancer sku to standard
# IMPORTANT!!! This is a one way action you can't downgrade to basic anymore
az aks update \
  --resource-group $RESOURCE_GROUP \
  --name $AKS_CLUSTER \
  --load-balancer-sku standard
```

## Configure AKS Cluster

### Install Kubectl

https://kubernetes.io/docs/tasks/tools/

### Install Helm Package Manager

https://helm.sh/docs/intro/install/

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
