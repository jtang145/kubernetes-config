# Running AKS for mssql

# Setups

## Setup on Windows Client

```
az account set -s 941c4202-4cd1-47bf-8f41-xxxxx
az login
az group create -l "Japan East" -n rg-xxxx

az aks create --resource-group rg-xxxx --name test-nodes --node-count 1 --node-vm-size Standard_B2s --node-osdisk-size 60 --generate-ssh-keys
```

## Setup on Linux Client

```

export AZURE_SUBSCRIPTION_ID=[to be filled]
export AZURE_RESOURCE_GROUP=rg-xxxx
export AZURE_REGION=[to be filled]

#
az account set -s $AZURE_SUBSCRIPTION_ID
az login
az group create -l $AZURE_REGION -n $AZURE_RESOURCE_GROUP

# 
export AKS_CLUSTER_NAME=test-nodes

az aks create --resource-group $AZURE_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --node-count 1 --node-vm-size Standard_B2s --node-osdisk-size 30 --generate-ssh-keys
```

# Check Setup

```
az aks get-credentials --resource-group rg-xxxx --name test-nodes
kubectl get nodes
```

# Operations

## Install MSSQL into kuberneters

```
kubectl apply -f SQLServer2019.yaml

kubectl get deployment
```

## Check MSSQL Status
Via sqlcmd client:
```
sqlcmd -S 52.175.127.143,31433 -U sa -Q 'SELECT @@VERSION' -P 'S0methingS@Str0ng!'
```
Via SSMS
Server Name: 52.175.127.143,31433
Authentication: SQL Server Authentication
Login: sa
Password: S0methingS@Str0ng! 