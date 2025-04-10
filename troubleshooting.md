


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

## Server 500 Error
servas-secret: 
base64:53lG5CvJmaK2Vx2ZJ6VOwsKEmYEdvmKkX+VD4UjJ1aU=


Check the servas:stats
```
php artisan servas:stats
```

Check the migration status
```
php artisan migrate:status
```
Force the migration
```
php artisan migrate --force
```
Check the app to database connectivity 
```
     kubectl exec -it <servas-pod-name> -n demo-project -- mysql -h mariadb -u <username> -p<password> -e "SHOW DATABASES;"
```

```

/app # cat .env
APP_NAME=servas
APP_ENV=production
APP_DEBUG=false
APP_URL=http://localhost
SERVAS_ENABLE_REGISTRATION=true
DB_CONNECTION=mysql
DB_HOST=mariadb
DB_PORT=3306
DB_DATABASE=servas
DB_USERNAME=servas
DB_PASSWORD=servas
MYSQL_ATTR_SSL_VERIFY_SERVER_CERT=false
DB_SSL=false
DB_SSL_VERIFY=false
APP_KEY=
/app # msql -h mariadb -u servas -pservas -e "SHOW DATABASES;"
/bin/sh: msql: not found
/app # which mariadb
/usr/bin/mariadb
/app # which msql
/app # mariadb -h mariadb -u servas -pservas -e "SHOW DATABASES;"
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
/app # 
```

     k exec --stdin --tty servas-685f558bcf-zg6tb -n demo-project -- mysql -h mariadb -u servas -pservas --skip-ssl -e "SHOW DATABASES;"