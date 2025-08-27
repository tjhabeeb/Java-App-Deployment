# Java-App-Deployment

This repository contains the setup and deployment instructions for the **Java-App-Deployment** project.  
It provisions a complete multi-VM application stack using **Vagrant + VirtualBox**, and deploys a Java web application with Tomcat, Nginx, RabbitMQ, Memcache, and MySQL.

---

## 🚀 Project Architecture

The stack consists of five VMs:

- **db01** → MySQL (Database)
- **mc01** → Memcache (Caching)
- **rmq01** → RabbitMQ (Message Broker)
- **app01** → Tomcat (Application Server + Java App)
- **web01** → Nginx (Web Server / Reverse Proxy)

---

## ⚙️ Prerequisites

Install the following on your host machine:

1. [Oracle VM VirtualBox](https://www.virtualbox.org/)
2. [Vagrant](https://developer.hashicorp.com/vagrant/downloads)
3. Vagrant plugins:
   ```bash
   vagrant plugin install vagrant-hostmanager
   ```
4. Git Bash (or any equivalent terminal)

---

## 📦 VM Setup

1. Clone this repository:
   ```bash
   git clone https://github.com/tjhabeeb/Java-App-Deployment.git
   cd Java-App-Deployment/vagrant/Automated_provisioning_WinMacIntel
   ```

2. Bring up the VMs:
   ```bash
   vagrant up
   ```

   > Note: Provisioning all VMs may take some time. If it stops midway, rerun `vagrant up`.

---

## 🔧 Manual Provisioning (if needed)

If you prefer manual setup, you can provision each service in this order:

1. **MySQL (db01)**
   ```bash
   vagrant ssh db01
   sudo dnf update -y
   sudo dnf install epel-release git mariadb-server -y
   sudo systemctl enable --now mariadb
   mysql_secure_installation
   # Set root password: admin123
   mysql -u root -padmin123 -e "create database accounts;"
   mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
   ```

2. **Memcache (mc01)**
   ```bash
   vagrant ssh mc01
   sudo dnf install epel-release memcached -y
   sudo systemctl enable --now memcached
   ```

3. **RabbitMQ (rmq01)**
   ```bash
   vagrant ssh rmq01
   sudo dnf install epel-release wget -y
   sudo dnf -y install centos-release-rabbitmq-38
   sudo dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
   sudo systemctl enable --now rabbitmq-server
   ```

4. **Tomcat + Java (app01)**
   ```bash
   vagrant ssh app01
   sudo dnf -y install java-17-openjdk java-17-openjdk-devel git wget
   cd /tmp
   wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
   tar xzvf apache-tomcat-10.1.26.tar.gz
   sudo cp -r apache-tomcat-10.1.26 /usr/local/tomcat
   sudo useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
   sudo chown -R tomcat:tomcat /usr/local/tomcat
   ```

   ### Build & Deploy App
   ```bash
   cd /tmp
   wget https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
   unzip apache-maven-3.9.9-bin.zip
   sudo cp -r apache-maven-3.9.9 /usr/local/maven
   export PATH=$PATH:/usr/local/maven/bin

   git clone -b local https://github.com/tjhabeeb/Java-App-Deployment.git
   cd Java-App-Deployment
   mvn install
   sudo systemctl stop tomcat
   sudo rm -rf /usr/local/tomcat/webapps/ROOT*
   sudo cp target/*.war /usr/local/tomcat/webapps/ROOT.war
   sudo systemctl start tomcat
   ```

5. **Nginx (web01)**
   ```bash
   vagrant ssh web01
   sudo apt update && sudo apt install nginx -y
   sudo tee /etc/nginx/sites-available/vproapp <<EOF
   upstream vproapp {
       server app01:8080;
   }
   server {
       listen 80;
       location / {
           proxy_pass http://vproapp;
       }
   }
   EOF
   sudo ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
   sudo systemctl restart nginx
   ```

---

## ✅ Accessing the App

After setup, open your browser and navigate to:

```
http://web01
```

This should serve the Java web application through Nginx → Tomcat → MySQL/Memcache/RabbitMQ.

---

## 📝 Notes

- If provisioning fails, you can reprovision a single VM:
  ```bash
  vagrant reload <vm-name> --provision
  ```
- If you want to destroy and rebuild:
  ```bash
  vagrant destroy -f
  vagrant up
  ```

---

## 📌 Project Name

**Repository:** [Java-App-Deployment](https://github.com/tjhabeeb/Java-App-Deployment)  
**Stack:** Java, Tomcat, Maven, MySQL, Memcache, RabbitMQ, Nginx  
