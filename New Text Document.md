## Manual Deploy CI/CD Infrastructure for Development Spring Boot Java Application at GCP using Compute Engine VMs
### Agenda
1. Enables APIs:
   - Compute Engine API  
   - Cloud SQL Admin API 
   - Cloud SQL 
2. Create Custom VPC
3. Create firewall rule with specified tag and open needed ports:
   - 22
   - 8080
   - 3306 
4. Create VMs (Jenkins master, Jenkins agent, App VM)
   - Create service accounts, add necessary roles
   - Create VMs with Linux Debian 10
   - (Network tab: choose network and set network tag) Apply firewall rule using network tag
5. Setup Jenkins master, Jenkins agent and App VMs
   - Install necessary packages for all VMs (git, wget, JDK8, Jenkins)
   - Install Jenkins plugins
   - Setup maven tool
6. Connecting Jenkins agent via SSH
   - Create a SSH key pair 
   - Add the Cloud Shell public SSH key to project's metadata
7. Setup CloudSQL instance
   - Create Cloud SQL instance with MySQL 5.7
   - Create database, user and password 
   - Add password to Secret Manager
8. Setup Cloud SQL Proxy on App VM
   - Install the Cloud SQL Proxy on the Compute Engine instance
   - Create file /etc/systemd/system/cloud-sql-proxy.service
   - Start Cloud SQL Proxy on system boot with Systemd
9. Runnig spring-boot petclinic App on target VM
   - Start spring-boot petclinic app on system boot with Systemd 
10. Create CI/CD pipeline using Jenkinsfile

## 8. Setup Cloud SQL Proxy on App VM

### Install the Cloud SQL Proxy on the Compute Engine instance

```
cd /usr/local/bin
sudo wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
sudo chmod +x cloud_sql_proxy
```
### Create file /etc/systemd/system/cloud-sql-proxy.service
Before creating this file you should fill `<INSTANCE_CONNECTION_NAME>`

```bash
[Unit]
Description=Connecting MySQL Client from Compute Engine using the Cloud SQL Proxy
Documentation=https://cloud.google.com/sql/docs/mysql/connect-compute-engine
Requires=networking.service
After=networking.service

[Service]
WorkingDirectory=/usr/local/bin
ExecStart=/usr/local/bin/cloud_sql_proxy -dir=/var/run/cloud-sql-proxy -instances=<INSTANCE_CONNECTION_NAME>=tcp:3306
Restart=always
StandardOutput=/var/log/cloud_sql_proxy-stdout.log
User=cloudsqlproxy

[Install]
WantedBy=multi-user.target
```
### Start Cloud SQL Proxy on system boot with Systemd

1. Copy cloud-sql-proxy.service to `/etc/systemd/system/cloud-sql-proxy.service`
2. Add user cloudsqlproxy `sudo useradd -r -s /bin/false cloudsqlproxy`
3. Change ownership `chown cloudsqlproxy:cloudsqlproxy /usr/local/bin/cloud_sql_proxy`
4. Run `sudo systemctl daemon-reload`
5. Enable autostart `sudo systemctl enable cloud-sql-proxy`
6. Start service `sudo systemctl start cloud-sql-proxy`

## 9. Runnig spring-boot petclinic App on target VM
### Start spring-boot petclinic app on system boot with Systemd

1. Add user petclinic `sudo useradd -r -s /bin/false petclinic`
2. Create app dir `sudo mkdir -p /opt/app`
3. Copy spring-petclinic app to `/opt/app`
4. Change ownership `chown petclinic:petclinic /opt/app/spring-petclinic-2.4.5.jar`
5. Create file /etc/systemd/system/petclinic.service

```bash
[Unit]
Description=Startting petclinic as service
Documentation=https://github.com/spring-projects/spring-petclinic
Requires=cloud-sql-proxy.service
After=cloud-sql-proxy.service

[Service]
WorkingDirectory=/opt/app
ExecStart=/bin/java -jar -Dspring.profiles.active=mysql spring-petclinic-2.4.5.jar
Restart=always
StandardOutput=journal
User=petclinic

[Install]
WantedBy=multi-user.target
```

6. Run `sudo systemctl daemon-reload`
7. Enable autostart `sudo systemctl enable petclinic`
8. Start service `sudo systemctl start petclinic`

