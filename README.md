# Deploying a Dynamic Sub-domain website for a Client - PayRoyal Using bash Scripting

1. **Creation of a Virtual Machine in AWS** 

![image.png](attachment:654064ed-9e89-4690-bbbf-fd473bb09baa:image.png)

**Login to the server using mobaXterm through ssh-key**

![image.png](attachment:92bfe079-e8c9-4ffb-b511-cc3e31de0e72:image.png)

**carefully write out the script**

```bash
#!/bin/bash

# Update system and install Docker & Docker Compose
sudo apt update -y 
sudo apt install docker.io docker-compose -y

# Create directories for MySQL configuration
mkdir -p mysql/{data,initdb,config}

# Create SQL initialization script
cat <<EOF > mysql/initdb/grant-all.sql
CREATE USER 'vincent2020'@'%' IDENTIFIED BY 'vincent@2020';
GRANT ALL PRIVILEGES ON *.* TO 'vincent2020'@'%' WITH GRANT OPTION;
CREATE USER 'wordpress'@'%' IDENTIFIED BY 'Wordpress@2020';
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;
CREATE DATABASE royalpaydb;
FLUSH PRIVILEGES;
EOF

# Create MySQL config file
cat <<EOF > mysql/config/mysqld.cnf
[mysqld]
user = mysql
port = 3306
datadir = /var/lib/mysql
bind-address = 0.0.0.0
mysqlx-bind-address = 0.0.0.0
key_buffer_size = 16M
myisam-recover-options = BACKUP
general_log_file = /var/log/mysql/query.log
general_log = 1
log_error = /var/log/mysql/error.log
max_binlog_size = 100M
EOF

# Create docker-compose file
cat <<EOF > mysql/docker-compose.yaml
version: '3.8'
services:
  mysql:
    image: mysql:latest
    container_name: mysql-container
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: Vincent2020
      MYSQL_DATABASE: royalpaydb
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: Wordpress@2020
    ports:
      - "3306:3306"
    volumes:
      - ./config:/etc/mysql/conf.d
      - ./data:/var/lib/mysql
      - ./initdb:/docker-entrypoint-initdb.d
    networks:
      - wp-net

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: unless-stopped
    depends_on:
      - mysql
    ports:
      - "8081:80"
    environment:
      WORDPRESS_DB_HOST: mysql-container:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: Wordpress@2020
      WORDPRESS_DB_NAME: royalpaydb
    networks:
      - wp-net

networks:
  wp-net:
    driver: bridge
EOF

# Move into the mysql directory to run docker-compose
cd mysql
sudo docker-compose up -d

```

create the script file using `nano [deploy-data@payroyal.sh](mailto:deploy-data@payroyal.sh)` (name of file)

inspect the file using `cat [deploy-data@payroyal.sh](mailto:deploy-data@payroyal.sh)` 

![image.png](attachment:ea43f8f0-fd6d-4637-9318-2bfe1d5d0482:image.png)

Then deploy using `sh [deploy-data@payroyal.sh](mailto:deploy-data@payroyal.sh)` 

confirm the Containers are running 

![image.png](attachment:562fbccb-01a9-436a-a023-7d171e3b7b98:image.png)

**Connect to Domain using Advance DNS Settings**

![image.png](attachment:b2979829-fecc-441e-90e1-450a805374d2:image.png)

 

**Confirm it has been Propagated using [dnschecker.org](http://dnschecker.org)** 

![image.png](attachment:a8e3ed80-1d9b-4b61-bdbf-6a5b343413ec:image.png)

**Access WordPress in your browser**

http://YOUR_IP:8081

[http://data.payroyal.online:8081](http://data.payroyal.online:8081/wp-admin/)

![image.png](attachment:10a7d09d-25cd-4460-bbfa-6c326d62539e:image.png)

![image.png](attachment:c09c796e-0a26-48e0-82ea-887edf78c051:image.png)

**Then Continue with the setups**

![image.png](attachment:a67acff1-7b7b-4e06-b62f-6e9ef3d618b4:image.png)
