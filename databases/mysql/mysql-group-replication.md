# Setup MySQL InnoDB cluster guide

## Setup StatefulSets

Follow the guide from https://github.com/wwwted/Oracle-Cloud/blob/master/kubernetes-mysql/k8s_innodb_cluster.md,
we can setup a innodb cluster. 

## Configurations

```
kubectl exec -it  mysql-innodb-cluster-0 -- mysql -uroot -p_MySQL2020_ -e"SET SQL_LOG_BIN=0; CREATE USER 'idcAdmin'@'%' IDENTIFIED BY 'idcAdmin'; GRANT ALL ON *.* TO 'idcAdmin'@'%' WITH GRANT OPTION";
kubectl exec -it  mysql-innodb-cluster-1 -- mysql -uroot -p_MySQL2020_ -e"SET SQL_LOG_BIN=0; CREATE USER 'idcAdmin'@'%' IDENTIFIED BY 'idcAdmin'; GRANT ALL ON *.* TO 'idcAdmin'@'%' WITH GRANT OPTION";
kubectl exec -it  mysql-innodb-cluster-2 -- mysql -uroot -p_MySQL2020_ -e"SET SQL_LOG_BIN=0; CREATE USER 'idcAdmin'@'%' IDENTIFIED BY 'idcAdmin'; GRANT ALL ON *.* TO 'idcAdmin'@'%' WITH GRANT OPTION";
```

Interaction
```
 shell.connect("idcAdmin@mysql-innodb-cluster-0:3306");
 cluster = dba.getCluster();
 
```