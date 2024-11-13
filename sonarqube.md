# sonarqube
sonarqube installation along with integration


Prerequisites
Need an EC2 instance (min t2.small)
Install Java-17
 apt-get update   
 apt  list | grep openjdk-17  
 apt-get install openjdk-17-jdk -y 


Install & Setup Postgres Database for SonarQube
Source: https://www.postgresql.org/download/linux/ubuntu/

Install Postgres database

sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > etc/apt/sources.list.d/pgdg.list'
sudo apt -y install postgresql-15  


Set a password and connect to database (setting password as "admin" password)
sudo passwd postgres
su - postgres


Create a database user and database for sonarque

createuser sonar
psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;

Restart postgres database to take latest changes effect

systemctl restart postgresql 
systemctl status postgresql


apt install net-tools
netstat -tunlp

You should see postgres is running on 5432


Source: https://docs.sonarqube.org/latest/requirements/requirements/

Added below entries in /etc/sysctl.conf

vm.max_map_count=524288
fs.file-max=131072
ulimit -n 131072
ulimit -u 8192

Add below entries in /etc/security/limits.conf
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
reboot the server
init 6


SonarQube Setup
Download soarnqube in /opt dir and extract it.

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.7.96285.zip
unzip sonarqube-9.9.7.96285.zip 


Update sonar.properties with below information

#sonar.jdbc.username=sonar
#sonar.jdbc.password=admin
#sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
#sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError


Create a `/etc/systemd/system/sonarqube.service` file start sonarqube service at the boot time 
  
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


Rename the sonarqube.9.9 directory to sonarqube in /opt dir

mv sonarqube.9.9.7.96285 sonarqube

Add sonar user and grant ownership to /opt/sonarqube directory
useradd -d /opt/sonarqube sonar
chown -R sonar:sonar /opt/sonarqube


Reload the demon and start sonarqube service
systemctl daemon-reload 
systemctl start sonarqube.service 
