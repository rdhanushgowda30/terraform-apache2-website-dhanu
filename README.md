# AWS VPC and EC2 Deployment Using Terraform

## Project Overview

This project provisions AWS infrastructure using Terraform and deploys an Apache web server on an EC2 instance using User Data.

The infrastructure is created using Infrastructure as Code (IaC), making the deployment repeatable and easier to manage.

## Architecture

The project creates the following AWS resources:

- **VPC:** Provides an isolated virtual network.
- **Public Subnet:** Hosts the EC2 instance.
- **Internet Gateway:** Enables internet connectivity for the VPC.
- **Route Table:** Routes outbound internet traffic through the Internet Gateway.
- **Route Table Association:** Associates the route table with the public subnet.
- **Security Group:** Allows inbound SSH (22) and HTTP (80) traffic.
- **EC2 Instance:** Runs the Apache web server.
- **User Data:** Installs and starts Apache automatically during instance initialization.

### Architecture Flow

```text
AWS Region: us-east-1
        |
        VPC (10.0.0.0/16)
        |
        Public Subnet (10.0.1.0/24)
        |
        +---- Internet Gateway
        |          |
        |     Route Table
        |     0.0.0.0/0
        |
        EC2 Instance
        |
        Security Group
        |-- SSH: 22
        |-- HTTP: 80
        |
        User Data
        |
        Install Apache
        |
        Display Web Page
```

## Prerequisites

Install and configure the following:

- An AWS account
- AWS CLI
- Terraform
- AWS credentials configured with appropriate permissions
- Git

Verify the installations:

```bash
terraform -version
aws --version
aws sts get-caller-identity
```

Configure AWS credentials if needed:

```bash
aws configure
```

Enter your AWS access key, secret access key, default region (`us-east-1`), and preferred output format when prompted. Do not commit credentials to GitHub.

## Project Structure

```text
terraform-vpc-ec2/
├── main.tf
├── .gitignore
└── README.md
```

- `main.tf` — Contains the AWS provider, networking resources, security group, EC2 instance, User Data, and output.
- `.gitignore` — Excludes Terraform state, local configuration, and sensitive files.
- `README.md` — Documents the project and deployment instructions.

## Terraform Resource Explanation

### 1. AWS Provider

```hcl
provider "aws" {
  region = "us-east-1"
}
```

Configures Terraform to create resources in the Northern Virginia AWS Region.

### 2. VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

Creates a VPC with a private IPv4 address range of `10.0.0.0/16`.

### 3. Public Subnet

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
}
```

Creates a subnet within the VPC and enables automatic public IPv4 address assignment for instances launched in it.

### 4. Internet Gateway and Routing

The Internet Gateway connects the VPC to the internet. The public route table contains a default route:

```hcl
route {
  cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.main.id
}
```

The route table association attaches this route table to the public subnet.

### 5. Security Group

The security group permits:

| Protocol | Port | Purpose |
|---|---:|---|
| TCP | 22 | SSH administration |
| TCP | 80 | HTTP web traffic |
| All outbound | All | Outbound connections |

For improved security, restrict SSH access to your own public IP address rather than allowing `0.0.0.0/0`.

### 6. EC2 Instance and User Data

The EC2 instance is launched in the public subnet and associated with the security group.

User Data automatically executes a bootstrap script during the instance's first boot. For an Ubuntu AMI, the script installs Apache, starts the service, enables it at boot, and creates a custom HTML page.

Example User Data for Ubuntu:

```bash
#!/bin/bash
apt-get update -y
apt-get install -y apache2
echo '<h1>Hello from Terraform</h1>' > /var/www/html/index.html
systemctl enable --now apache2
```

**Note:** The AMI ID must correspond to Ubuntu for these commands to work. Amazon Linux uses a different package manager and Apache service name.

## Deployment Instructions

### Step 1: Clone the Repository

Replace the URL with your GitHub repository URL.

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
cd YOUR-REPOSITORY
```

### Step 2: Initialize Terraform

```bash
terraform init
```

Downloads the required provider plugins and initializes the working directory.

### Step 3: Format the Configuration

```bash
terraform fmt
```

Formats Terraform files using standard formatting conventions.

### Step 4: Validate the Configuration

```bash
terraform validate
```

Checks the Terraform configuration for syntax and configuration errors.

### Step 5: Review the Execution Plan

```bash
terraform plan
```

Shows the resources Terraform intends to create, update, or delete.

### Step 6: Create the Infrastructure

```bash
terraform apply
```

Review the proposed changes and type `yes` when prompted.

Terraform creates the VPC, subnet, Internet Gateway, route table, security group, and EC2 instance.

### Step 7: Get the Public IP

The configuration exports the instance's public IPv4 address.

```bash
terraform output instance_public_ip
```

Wait for cloud-init to finish installing Apache before testing the website.

### Step 8: Access the Web Server

Open the following URL in a browser, replacing the placeholder with the actual public IP:

```text
http://<EC2-PUBLIC-IP>
```

Expected page:

**Hello from Terraform**

## Troubleshooting

### Apache is not running

Connect to the EC2 instance and check the service:

```bash
sudo systemctl status apache2
```

Inspect User Data output:

```bash
sudo cat /var/log/cloud-init-output.log
```

Check whether the web server responds locally:

```bash
curl http://localhost
```

For Ubuntu, install Apache manually if necessary:

```bash
sudo apt-get update -y
sudo apt-get install -y apache2
sudo systemctl enable --now apache2
```

### The website does not load

Check the following:

- The EC2 instance has a public IPv4 address.
- The subnet is associated with the route table containing `0.0.0.0/0` through the Internet Gateway.
- The security group permits inbound TCP port 80.
- Apache is running.
- Network ACL rules allow the required inbound and outbound traffic.
- The instance has internet connectivity to download packages.

### User Data changes are not reflected

User Data normally runs on the initial boot. Changing the script in Terraform does not guarantee that it will run again on an existing instance. Review the Terraform plan and replace the EC2 instance if necessary.

## Cleanup

To avoid ongoing AWS charges, remove the resources created by this project when finished:

```bash
terraform destroy
```

Review the deletion plan and type `yes` when prompted.

**Warning:** This deletes the Terraform-managed infrastructure, including the EC2 instance, VPC, subnet, and networking resources.

## Key Learnings

- Provisioning AWS infrastructure with Terraform.
- Creating VPCs, subnets, Internet Gateways, and route tables.
- Configuring EC2 security groups.
- Launching EC2 instances in public subnets.
- Automating software installation using EC2 User Data.
- Deploying a basic Apache web server.
- Using Terraform initialization, formatting, validation, planning, applying, and destroying commands.

## Technologies Used

- AWS
- Terraform
- Amazon EC2
- Amazon VPC
- Linux
- Apache HTTP Server
- Bash
- Git and GitHub

## Author

**Dhanush R**

Cloud & DevOps Engineering | AWS | Terraform | Linux | CI/CD