_Note: This was my first application converted from docker-compose to helm charts used as a learning opportunity for Kubernetes skills development_


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
base64:<secret-long-string>


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

     k exec --stdin --tty servas-<pod-name> -n demo-project -- mysql -h mariadb -u servas -pservas --skip-ssl -e "SHOW DATABASES;"

```

```
# Apply the changes
kubectl apply -f single-cluster/servas/templates/deployment.yaml

# Restart the deployment
kubectl rollout restart deployment servas -n demo-project

# Monitor the logs
kubectl logs -f $(kubectl get pod -l app=servas -n demo-project -o jsonpath='{.items[0].metadata.name}') -n demo-project
```

# 🎉 Great News - Application is Working!

Let's document what fixed the issues for future reference:

### Key Changes that Resolved the Problems:

1. **Database Configuration**
   - Disabled SSL requirements for MariaDB
   - Added proper database connection environment variables

2. **Environment Setup**
   - Created dynamic `.env` file with all required variables 
   _Note: Need to remove this as the volume mount where the file is written is not used_
   - Set correct file permissions (644)
   - Included `SESSION_DRIVER=file` setting

3. **Database Migration**
   - Successfully ran migrations with `--force` flag
   - Schema loaded correctly after SSL configuration

4. **Environment Variables**
   - Added all necessary Laravel and MySQL configuration variables
   - Properly configured SSL-related settings
   - Added Laravel schema loading variables

### Next Steps (Optional):

1. **Backup Current Configuration**
```bash
# Export working deployment configuration
kubectl get deployment servas -n demo-project -o yaml > working-deployment-backup.yaml
```

2. **Monitor Application**
```bash
# Watch application logs
kubectl logs -f -l app=servas -n demo-project

# Check pod status
kubectl get pods -n demo-project -l app=servas
```

3. **Consider Setting Up**
- Secure the application properly  - TLS 
- Secure the containers to run as non-root
- Health checks
- Resource limits
- Horizontal Pod Autoscaling (HPA)
- Regular database backups


# Fix Laravel Database Migration and Composer Issues

The logs show two main issues:
1. `composer` command not found
2. Duplicate entry error during migrations

Let's update the deployment to fix these issues:

````yaml
spec:
  template:
    spec:
      containers:
        - name: servas
          image: beromir/servas:latest
          imagePullPolicy: IfNotPresent
          command:
            - /bin/sh
            - -c
            - |
              # Create directories and set permissions
              mkdir -p /app/storage/logs
              chmod -R 775 /app/storage
              
              # Enable error logging
              echo "error_reporting = E_ALL" > /usr/local/etc/php/conf.d/error-logging.ini
              echo "log_errors = On" >> /usr/local/etc/php/conf.d/error-logging.ini
              echo "error_log = /app/storage/logs/php_errors.log" >> /usr/local/etc/php/conf.d/error-logging.ini
              echo "display_errors = On" >> /usr/local/etc/php/conf.d/error-logging.ini
              
              # Disable MySQL SSL requirement (MariaDB syntax)
              cat << EOF > /etc/my.cnf
              [client]
              ssl=0
              ssl-verify-server-cert=0
              EOF
              
              # Write environment variables to .env file
              cat << EOF > /app/.env
              APP_KEY="${APP_KEY}"
              APP_NAME="${APP_NAME}"
              APP_ENV="${APP_ENV}"
              APP_DEBUG="${APP_DEBUG}"
              APP_URL="${APP_URL}"
              SERVAS_ENABLE_REGISTRATION="${SERVAS_ENABLE_REGISTRATION}"
              DB_CONNECTION="${DB_CONNECTION}"
              DB_HOST="${DB_HOST}"
              DB_PORT="${DB_PORT}"
              DB_DATABASE="${DB_DATABASE}"
              DB_USERNAME="${DB_USERNAME}"
              DB_PASSWORD="${DB_PASSWORD}"
              MYSQL_ATTR_SSL_VERIFY_SERVER_CERT="${MYSQL_ATTR_SSL_VERIFY_SERVER_CERT}"
              DB_SSL="${DB_SSL}"
              DB_SSL_VERIFY="${DB_SSL_VERIFY}"
              SESSION_DRIVER=file
              EOF
              
              # Set proper permissions
              chmod 644 /app/.env
              
              # Install composer if not present
              if ! command -v composer &> /dev/null; then
                curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
              fi
              
              # Clear Laravel caches first
              php artisan config:clear
              php artisan cache:clear
              php artisan route:clear
              php artisan view:clear
              
              # Drop all tables and run fresh migrations
              php artisan db:wipe --force
              php artisan migrate --force
              php artisan db:seed --force
              
              # Start Laravel
              php artisan serve --host=0.0.0.0 --port=80 --verbose
````

### Key changes:
1. Added composer installation
2. Replaced `migrate:fresh` with explicit steps:
   - `db:wipe` to clean the database
   - `migrate` to run migrations
   - `db:seed` to seed the database
3. Removed Faker installation since it should be part of the project dependencies

After making these changes:

```bash
# Apply the changes
kubectl apply -f single-cluster/servas/templates/deployment.yaml

# Restart the deployment
kubectl rollout restart deployment servas -n demo-project

# Monitor the logs
kubectl logs -f $(kubectl get pod -l app=servas -n demo-project -o jsonpath='{.items[0].metadata.name}') -n demo-project
```

This should resolve both the composer installation issue and the migration errors.