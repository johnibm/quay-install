# Create an internal registry using quay (test purposes only) #
## Reference: ##
#### https://access.redhat.com/documentation/en-us/red_hat_quay/3.8/html/deploy_red_hat_quay_for_proof-of-concept_non-production_purposes/poc-getting-started ####
## Install tools and login to registry.redhat.io ##
```
sudo yum install -y container-tools
sudo podman login registry.redhat.io
```

## Configure Firewall ##
```
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=5432/tcp
firewall-cmd --permanent --add-port=5433/tcp
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --reload
```
## Configure Postgres ##
```
export QUAY=/home/quay
mkdir -p $QUAY/postgres-quay
setfacl -m u:26:-wx $QUAY/postgres-quay
sudo podman run -d --rm --name postgresql-quay \
-e POSTGRESQL_USER=quayuser \
-e POSTGRESQL_PASSWORD=quaypass \
-e POSTGRESQL_DATABASE=quay \
-e POSTGRESQL_ADMIN_PASSWORD=adminpass \
-p 5432:5432 \
-v $QUAY/postgres-quay:/var/lib/pgsql/data:Z \
registry.redhat.io/rhel8/postgresql-13:1-109
sudo podman exec -it postgresql-quay /bin/bash -c 'echo "CREATE EXTENSION IF NOT EXISTS pg_trgm" | psql -d quay -U postgres'
```

## Configure Redis
```
sudo podman run -d --rm --name redis \
-p 6379:6379 \
-e REDIS_PASSWORD=strongpassword \
registry.redhat.io/rhel8/redis-6:1-110
```

## Configure Quay
```
sudo podman run --rm -it --name quay_config -p 80:8080 -p 443:8443 registry.redhat.io/quay/quay-rhel8:v3.8.7 config secret
```

## Configure Quay via Web Interface ##

## Prepare Configuration Folder ##
```
mkdir $QUAY/config
cp /home/quay/quay-config.tar.gz $QUAY/config
cd $QUAY/config