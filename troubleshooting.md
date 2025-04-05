


## Blow it away and Start over

```
kubectl delete gitrepo servas -n fleet-local && kubectl apply -f fleet-servas-mariadb.yaml

```

## Use MiniKube ingress 
```
kubectl port-forward -n demo-project svc/servas 8080:80

```

## Check connectivity from app to db
To check connectivity from inside your app container to the MariaDB pod on port 3306, you can use the nc -z <mariadb_service_ip> 3306 command, where <mariadb_service_ip> is the IP address of your MariaDB service. 
```
nc -z 10.109.111.48 3360

```