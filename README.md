**Java Application Deployment with Reverse Proxy on AWS**

This project demonstrates how to deploy a Java-based Student Registration Web Application on AWS using Apache Tomcat, Amazon RDS (MySQL), and Apache HTTPD as a reverse proxy to securely route traffic.

Project Objective

- Deploy a .war Java application using Apache Tomcat on an EC2 instance.
- Integrate the application with an Amazon RDS MySQL database.
- Configure a second EC2 instance as a secure reverse proxy using Apache HTTPD.
- Ensure secure access: the application is only accessible via the proxy.

Prerequisites

- AWS Account with EC2 and RDS access
- SSH Key Pair to access EC2 instances
- Basic understanding of:
  - Linux terminal
  - Apache Tomcat
  - MySQL
  - Apache HTTPD (or reverse proxy setup)
- Files Required:
  - student.war (Java web application)
  - mysql-connector-java-8.x.x.jar (JDBC driver)

Architecture Overview

User → Proxy EC2 (Apache HTTPD, port 80)
             ↓
     Backend EC2 (Tomcat, port 8080)
             ↓
        Amazon RDS (MySQL)

Technologies Used

- AWS EC2 (2 instances)
- Amazon RDS (MySQL)
- Apache Tomcat 9
- Apache HTTPD (Reverse Proxy)
- MySQL Connector/J
- Java 8
- Security Groups & Private Networking

Setup Instructions

1. EC2 Infrastructure Setup

- Launch two EC2 Instances:
  - Backend-EC2: For Java application and Tomcat
  - Proxy-EC2: For Apache HTTPD reverse proxy

- Security Groups:
  - Backend-EC2:
    - Allow: TCP 22 (SSH), TCP 8080 only from Proxy-EC2’s private IP
  - Proxy-EC2:
    - Allow: TCP 22 (SSH), TCP 80 (HTTP) from 0.0.0.0/0

2. Java Application Deployment (on Backend-EC2)

- Install Java & Tomcat:
  sudo yum update -y
  sudo yum install java-1.8.0-openjdk -y
  wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.xx/bin/apache-tomcat-9.0.xx.tar.gz
  tar -xvzf apache-tomcat-9.0.xx.tar.gz
  mv apache-tomcat-9.0.xx tomcat

- Deploy WAR file:
  cp student.war /home/ec2-user/tomcat/webapps/

- Add MySQL Connector:
  cp mysql-connector-java-8.x.x.jar /home/ec2-user/tomcat/lib/

- Start Tomcat:
  cd /home/ec2-user/tomcat/bin
  ./startup.sh

3. Database Setup (Amazon RDS MySQL)

- Create RDS Instance:
  - Engine: MySQL
  - Public Access: No
  - DB Name: studentdb

- Create Table:
  CREATE TABLE registration (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    course VARCHAR(100)
  );

- Update DB credentials in Java app:
  - Hostname: RDS Endpoint
  - Port: 3306
  - Username/Password: As created in RDS
  - Database: studentdb

4. Reverse Proxy Setup (on Proxy-EC2)

- Install Apache HTTPD:
  sudo yum install httpd -y
  sudo systemctl start httpd
  sudo systemctl enable httpd

- Configure Proxy:
  sudo vi /etc/httpd/conf.d/proxy.conf

- Add this configuration:
  <VirtualHost *:80>
      ProxyPreserveHost On
      ProxyPass /student http://<Backend-Private-IP>:8080/student
      ProxyPassReverse /student http://<Backend-Private-IP>:8080/student

      ErrorLog /var/log/httpd/proxy_error.log
      CustomLog /var/log/httpd/proxy_access.log combined
  </VirtualHost>

- Restart Apache:
  sudo systemctl restart httpd

Security Setup

- Backend-EC2 is private: Only accepts traffic on port 8080 from Proxy-EC2’s private IP.
- Proxy-EC2 is public: Exposes HTTP on port 80.
- Direct access to Backend-EC2 is blocked from the internet.

Access URL

Access the web app via proxy:
http://<Proxy-EC2-Public-IP>/student

Summary

- Traffic Flow: User → Proxy (Apache HTTPD) → Backend (Tomcat) → RDS (MySQL)
- Deployment: Java WAR on Tomcat, RDS integration, Reverse Proxy setup
- Security: Backend is private, proxy-only access
