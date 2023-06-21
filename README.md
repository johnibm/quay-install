# Create an internal registry using quay (test purposes only) #
## Install tools and login to registry.redhat.io ##
```
sudo yum module install -y container-tools
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
## Setup Postgres ##
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

