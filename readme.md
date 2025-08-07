# 🛡️ Automated Backup of EC2 and RDS using AWS Backup

## 📌 Project Overview

This project demonstrates how to **automate backup and recovery** of AWS services using **AWS Backup**. We'll create and configure a centralized backup plan to protect:

- An **EC2 instance** (with a web server and test file)
- An **RDS MySQL database** (with sample data)

The goal is to ensure regular, reliable backups with minimal manual effort — a key requirement in real-world production environments.

---

## 📚 Table of Contents

1. [Infrastructure Setup](#1-infrastructure-setup)
   - Launch EC2
   - Launch RDS
2. [Install Web Server & Add Test Data](#2-install-web-server--add-test-data)
3. [Create and Insert Data in RDS](#3-create-and-insert-data-in-rds)
4. [Set Up AWS Backup](#4-set-up-aws-backup)
   - Backup Vault
   - Backup Plan
   - Assign Resources
5. [Trigger Backup and Validate](#5-trigger-backup-and-validate)
6. [View Recovery Points](#6-view-recovery-points)

---

## ✅ 1. Infrastructure Setup

### 🔸 Launch EC2 Instance

- Go to **EC2 > Launch Instance**
- Name: `backup-ec2`
- AMI: Amazon Linux 2 (or Ubuntu)
- Instance type: `t2.micro` (Free Tier)
- Key pair: Create or use existing `.pem` file
- Security Group: Allow:
  - Port **22** (SSH)
  - Port **80** (HTTP)
- Click **Launch Instance**

---

### 🔸 Launch RDS Instance (MySQL)

- Go to **RDS > Databases > Create database**
- Engine: **MySQL**
- Template: **Free Tier**
- DB Instance ID: `backupdb`
- Master username: `admin`
- Master password: `Admin12345!` *(Use your own strong password)*
- DB class: `db.t3.micro`
- Storage: 20GB (General Purpose SSD)
- Enable **Public access**
- Select the same VPC as your EC2
- Security Group: Allow port **3306**
- Click **Create database**
- Wait until status is **Available**

---

## ✅ 2. Install Web Server & Add Test Data

### 🖥️ Connect to EC2 via SSH

    ssh -i "your-key.pem" ec2-user@<EC2-PUBLIC-IP>

### 🛠️ Install Apache (Amazon Linux 2)

    sudo yum update -y
    sudo yum install httpd -y
    sudo systemctl start httpd
    sudo systemctl enable httpd

### 🌐 Add Sample HTML Page

    echo "Welcome to EC2 Backup Page" | sudo tee /var/www/html/index.html

- Open your EC2 public IP in browser
   → hhtp://ec2-public-ip
- We Should see: "Welcome to EC2 Backup Page"

---

## ✅ 3. Create and Insert Data in RDS

### 📦 Install MySQL Client (on EC2)
    sudo yum install mysql -y


### 🔌 Connect to RDS
    mysql -h <RDS-endpoint> -u admin -p

- Enter your password (Admin12345! or your custom one)

## 🗃️ Create DB and Insert Sample Data
    CREATE DATABASE backupdb;
    USE backupdb;

    CREATE TABLE testdata (
    id INT PRIMARY KEY,
    name VARCHAR(50)
    );

    INSERT INTO testdata VALUES (1, 'EC2 Backup'), (2, 'RDS Backup');

    SELECT * FROM testdata;

- ✅ Output should show two rows inserted.

---


## ✅ 4. Set Up AWS Backup

### 🔸 Create Backup Vault (Optional)
- Go to AWS Backup > Backup vaults > Create vault
- Go to AWS Backup > Backup vaults > Create vault
- Encryption: Default AWS KMS key
- Click Create

### 🔸 Create Backup Plan
- Go to AWS Backup > Backup Plans
- Click Create Backup Plan
- Choose: Build a new plan
- Plan Name: EC2-RDS-BackupPlan
- Click Add Backup Rule

### 🔸 Rule Settings:
- Rule name: DailyBackup
- Vault: MyBackupVault (or Default)
- Frequency: Daily
- Retention: 7 days
- Backup window: Default
- Transition to cold storage: (Optional)

### 🔸 Assign EC2 and RDS Resources
- Go to the plan → Click Assign Resources
- Assignment name: EC2-RDS-Assign
- IAM role: Use AWS default
- Assign by: Resource ID

- Select:

    - Your EC2 instance
    - Your RDS instance

- Click Assign resources

---

## ✅ 5. Trigger Backup and Validate

### 🔹 Start On-Demand Backup
- Go to Backup Plans
- Click your plan EC2-RDS-BackupPlan
- Click Actions > Start on-demand backup
- Choose both EC2 and RDS resources
- Click Start Backup

✅ Monitor the backup under Jobs

---

## ✅ 6. View Recovery Points

- Go to Backup Vaults
- Click your vault (e.g., MyBackupVault)

- Check Recovery Points

    - You should see separate entries for:

        - EC2 instance
        - RDS database
    - Status: Completed

### 🔹 Conclusion:
- This project ensures regular, automated backups for EC2 and RDS resources using AWS Backup, allowing for simple recovery in case of data loss or failure.

---

## 👤 Author

**Kishor Borse**  
💼 [LinkedIn Profile](https://www.linkedin.com/in/kishorborse/) 

