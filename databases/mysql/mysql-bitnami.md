# Setup MySQL cluster with bitnami 

## Resources

* Bitnami MySQL image is: https://hub.docker.com/r/bitnami/mysql
* Helm charts: https://hub.kubeapps.com/charts/bitnami/mysql

## Setup by helm
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql1 bitnami/mysql
```

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
* primary.service.type=LoadBalancer is needed for provision an external IP.


