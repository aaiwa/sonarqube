# Sonarqube Setup
sonarqube installation along with integration

## Prerequisites
Need an AWS EC2 instance (min t2.small)
`Install Java-17`
 ```sh
 apt-get update   
 apt  list | grep openjdk-17  
 apt-get install openjdk-17-jdk -y
``` 

## Install & Setup Postgres Database for SonarQube
Source: https://www.postgresql.org/download/linux/ubuntu/

## Install Postgres database
`Import the repository signing key:`
```sh
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```

`Create the repository configuration file:`
```sh
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > etc/apt/sources.list.d/pgdg.list'
```

`Update the package lists:`
```sh
sudo apt update
```

`Install the latest version of PostgreSQL:`
If you want a specific version, use 'postgresql-16' or similar instead of 'postgresql' in my case i am installing postgresql-15
```sh
sudo apt -y install postgresql-15  
```

`Set a password and connect to database (setting password as "admin" password)`

```sh
sudo passwd postgres
su - postgres
```

`Create a database user and database for sonarque`
```sh
createuser sonar
psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
```
`Restart postgres database to take latest changes effect`
```sh
systemctl restart postgresql 
systemctl status postgresql
```
'check the port number we need to first install the net-tools'
```sh
apt install net-tools
netstat -tunlp
```
You should see postgres is running on 5432


`Source: https://docs.sonarqube.org/latest/requirements/requirements/`

`Added below entries in /etc/sysctl.conf`
```sh
vm.max_map_count=524288
fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
```

`Add below entries in /etc/security/limits.conf`
```sh
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```
`reboot the server`
```sh
init 6
```
=============================================

# SonarQube Setup
Download [soarnqube](https://www.sonarqube.org/downloads/) in /opt dir and extract it.
```sh
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.7.96285.zip
unzip sonarqube-9.9.7.96285.zip 
```

`Update sonar.properties with below information`
```sh
#sonar.jdbc.username=sonar
#sonar.jdbc.password=admin
#sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
#sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError
```

`Create a `/etc/systemd/system/sonarqube.service` file start sonarqube service at the boot time `
```sh  
cat >> /etc/systemd/system/sonarqube.service <<EOL
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
User=sonar
Group=sonar
PermissionsStartOnly=true
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start 
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
StandardOutput=syslog
LimitNOFILE=65536
LimitNPROC=4096
TimeoutStartSec=5
Restart=always

[Install]
WantedBy=multi-user.target
EOL
```

`Rename the sonarqube.9.9 directory to sonarqube in /opt dir`
```sh
mv sonarqube.9.9.7.96285 sonarqube
```

`Add sonar user and grant ownership to /opt/sonarqube directory`
```sh
useradd -d /opt/sonarqube sonar
chown -R sonar:sonar /opt/sonarqube
```

`Reload the demon and start sonarqube service`
```sh
systemctl daemon-reload 
systemctl start sonarqube.service
```


# CleanUp
Stop sonarqube services and delete the instance

# Unable to access Sonarqube from browser?
1.Make sure port 9000 is opened at security group leave

2.start sonar service as a sonar user

3.user correct database credentials in the sonar.properties

4.use instance which has atleast 2 GB of RAM
