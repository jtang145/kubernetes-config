# Setup MySQL cluster with Bitnami 

## Resources

* Bitnami MySQL image is: https://hub.docker.com/r/bitnami/mysql
* Helm charts: https://hub.kubeapps.com/charts/bitnami/mysql
                                          
## Helm charts setup
Install Bitnami mysql chart:  
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql1 bitnami/mysql
```

## Setup MySQL classic replication by helm

For better tuned cluster, helm command may like this:
```
helm install mysql2 bitnami/mysql 
    --set auth.rootPassword=Rdis2fun!
    --set auth.database=app_database
    --set architecture=replication
    --set secondary.replicaCount=2
    --set auth.replicationPassword=abcd1234
    --set primary.service.type=LoadBalancer

# copy for run:
# helm install mysql1 bitnami/mysql --set auth.rootPassword=Rdis2fun!,auth.database=app_database,architecture=replication,secondary.replicaCount=2,auth.replicationPassword=abcd1234,primary.service.type=LoadBalancer
```


Note:
* Primary.service.type=LoadBalancer is needed for provision an external IP.
* Base on recent testing, the MySQL replication is only classic replication, not group replication yet.

## Setup standalone MySQL server instance
We can setup standalone mysql instance with specific versions, with the "image.tag" parameter, 
according to current Bitnami document, below versions are supported:
* 8.0
* 8.0.23 
* 5.7
* 5.7.33

sample commands:
```
helm install mysql_5.7 bitnami/mysql 
    --set image.tag=5.7.33
    --set auth.rootPassword=Rdis2fun! 
    --set primary.service.type=LoadBalancer
    
# copy for run:
# helm install mysql5733 bitnami/mysql --set image.tag=5.7.33,auth.rootPassword=Rdis2fun,primary.service.type=LoadBalancer
```
                                      
# Operation

## Get service public IP
```
kubectl get service mysql5733 -o jsonpath='{.status.loadBalancer.ingress[].ip}'
```


