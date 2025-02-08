# AWS 3-Tier Web Application

This repository contains the code and setup instructions for deploying a **3-tier web application** on AWS. The architecture includes:

- **Presentation Layer**: Frontend (React/HTML+CSS+JS) hosted on an EC2 instance or S3 with CloudFront.
- **Application Layer**: Backend API (Node.js/Python/Flask) running on a private EC2 instance.
- **Database Layer**: Amazon RDS (MySQL/PostgreSQL) for data storage.

---

## 🏗 Architecture Overview

- **Frontend (Public EC2 / S3 + CloudFront)**: Serves static content.
- **Backend (Private EC2, Security Group restricted to Frontend IPs)**: Handles API requests.
- **Database (RDS Instance, only accessible from Backend EC2)**.

---

## 🚀 Deployment Steps

### 1️⃣ Steps to be Done in AWS Console

#### **Networking Setup**
1. **Create a VPC**
   - Go to **VPC** > **Create VPC**.
   - Name: `myVPC`, CIDR: `172.20.0.0/20`.
![VPC SETUP](https://github.com/user-attachments/assets/23a7a043-6bd3-4ea5-8841-36df320712e5)

2. **Create Public Subnets** (one per AZ)
   - `my-public-web-subnet-1` (AZ: `us-east-2a`, CIDR: `172.20.1.0/24`)
   - `my-public-web-subnet-2` (AZ: `us-east-2b`, CIDR: `172.20.2.0/24`)
   - `my-public-web-subnet-3` (AZ: `us-east-2c`, CIDR: `172.20.3.0/24`)

3. **Create Private App Subnets**
   - `my-private-app-subnet-1` (AZ: `us-east-2a`, CIDR: `172.20.4.0/24`)
   - `my-private-app-subnet-2` (AZ: `us-east-2b`, CIDR: `172.20.5.0/24`)
   - `my-private-app-subnet-3` (AZ: `us-east-2c`, CIDR: `172.20.6.0/24`)

4. **Create Private DB Subnets**
   - `my-private-db-subnet-1` (AZ: `us-east-2a`, CIDR: `172.20.7.0/24`)
   - `my-private-db-subnet-2` (AZ: `us-east-2b`, CIDR: `172.20.8.0/24`)
   - `my-private-db-subnet-3` (AZ: `us-east-2c`, CIDR: `172.20.9.0/24`)
   - ![SUBNETS](https://github.com/user-attachments/assets/369a61a3-11cf-4c3e-bcd2-ddfe9788e08f)

5. **Create Route Tables**
   - `my-public-route-table`: Associate with public subnets.
   - `my-private-app-route-table`: Associate with app subnets.
   - `my-private-db-route-table`: Associate with DB subnets.
   - ![Route setup](https://github.com/user-attachments/assets/d34b3973-547c-4066-8fce-f44b68eb5e57)

6. **Create an Internet Gateway**
   - Name: `my-internet-gateway` > Attach to `myVPC`.
   - Update the public route table to direct `0.0.0.0/0` to the Internet Gateway.
   - ![Internet Gateway](https://github.com/user-attachments/assets/aa71e075-7b4a-4f1f-b5ae-cb5cc3547a6d)

7. **Create a NAT Gateway**
   - Create in `my-public-web-subnet-1`.
   - Allocate an Elastic IP.
   - Update private route tables to direct `0.0.0.0/0` to the NAT Gateway.
   - ![Nat setup](https://github.com/user-attachments/assets/7a4e0725-a561-421b-9f90-58d94b35aca8)

#### **Application Setup**
8. **Create EC2 Instances**
   - **Jump Server** (`my-jump-server`) in `my-public-web-subnet-1`.
   - **Backend Servers**
     - `my-php-app-server-1` in `my-private-app-subnet-1`
     - `my-php-app-server-2` in `my-private-app-subnet-2`

9. **Create Security Groups**
   - `my-jump-server-sg`: Allow SSH from your IP.
   - `my-php-app-server-sg`: Allow HTTP/HTTPS from ALB, SSH from Jump Server.
   - `my-db-sg`: Allow MySQL traffic from `my-php-app-server-sg`.

10. **Install Apache and PHP on App Servers**
    ```sh
    sudo yum update -y
    sudo yum install -y httpd php php-mysqlnd
    sudo systemctl enable httpd
    sudo systemctl start httpd
    ```
    ![Connect to your local pc](https://github.com/user-attachments/assets/02c3d4e9-16f0-4850-bdc5-95a05cf08aa0)

#### **Load Balancer Setup**
11. **Create an Application Load Balancer (ALB)**
    - Name: `my-alb` (Internet-facing)
    - Public Subnets: Attach all public subnets.
    - Security Group: `my-alb-sg` (Allow HTTP/HTTPS from `0.0.0.0/0`)
    - Create Target Group: `my-alb-php-tg` (Register backend EC2 instances)
    - Attach Target Group to ALB Listener on Port `80`
    - ![ALB setup](https://github.com/user-attachments/assets/04207529-7abb-41cf-89d2-8edc7d1f949f)

#### **Database Setup**
12. **Create an RDS Instance**
    - Engine: MySQL (Free Tier)
    - Instance: `t2.micro`
    - DB Subnet Group: Include all private DB subnets.
    - Security Group: `my-db-sg`
    - Save Endpoint for App Server Configuration.
    -  ![DB subnet group setup](https://github.com/user-attachments/assets/363c5496-a1ff-4689-881c-93951d80d0fc)
    -  ![Database setup](https://github.com/user-attachments/assets/e5e4f709-502b-47d0-a78e-7480a1a4ec4d)


13. **Update App Server Config**
    ```sh
    sudo nano /var/www/html/config.php
    # Update DB_HOST to RDS Endpoint
    ```

#### **Testing the Application**
14. **Verify Load Balancer Access**
    - Copy ALB DNS and test in browser.

15. **Enable Session Stickiness for ALB**
    - Target Group > Attributes > Enable Session Stickiness.

---

### 2️⃣ CI/CD Deployment (GitHub Actions Optional)
(Refer to previous section for automation setup.)

---

## 📌 Additional Enhancements
- Use **Docker** for containerized deployments.
- Implement **Terraform** or **CloudFormation** for IaC.

---

## 📞 Support
For any issues, feel free to open a GitHub **Issue** or reach out via email.

---

🔗 **GitHub Repository**: [(https://github.com/SREEGEETHES)]

📜 **License**: MIT
