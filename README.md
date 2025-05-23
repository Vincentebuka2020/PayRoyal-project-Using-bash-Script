# Deploying a Dynamic Sub-domain website for a Client - PayRoyal Using bash Scripting

1. **Creation of a Virtual Machine in AWS** 

![alt text](images/Vm.png)

**Login to the server using mobaXterm through ssh-key**

![alt text](images/ssh.png)

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


Then deploy using `sh [deploy-data@payroyal.sh](mailto:deploy-data@payroyal.sh)` 

confirm the Containers are running 

images/containers.png

**Connect to Domain using Advance DNS Settings**

![alt text](<images/dns record.png>)

 
**Confirm it has been Propagated using [dnschecker.org](http://dnschecker.org)** 

![alt text](images/dnschecker.png)

**Access WordPress in your browser**

http://YOUR_IP:8081

[http://data.payroyal.online:8081](http://data.payroyal.online:8081/wp-admin/)

![alt text](images/livewebsite.png)

**Then Continue with the setups**

![alt text](images/setuppage.png)
