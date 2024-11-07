---
title: Getting Started with Terraform- Multi-Resource Creation and Implicit Dependencies
# description: Use Jekyll to build a static website and host it on GitHub for free.
author: sharanya
date: 2024-11-04 17:45:00 +0800
categories: [Terraform]
tags: [Terraform, AWS, IaC, multi-resource deployment]
image:
  path: C:\Users\dci-student\Documents\portfolio\repo\sharanyagprasad.github.io\assets\img\Terraform\Terraform.png
  alt: Terraform logo

---



In this post, we’ll explore how to create multiple AWS resources with Terraform and understand the concept of implicit dependency. We'll walk through setting up a Virtual Private Cloud (VPC) along with subnets, internet gateways, security groups, and an EC2 instance.

By the end of this post, you’ll be able to:
1. Understand basic Terraform concepts.
2. Build AWS infrastructure using Terraform.
3. Recognize how implicit dependencies work in Terraform.
4. Use essential Terraform commands to manage infrastructure.

---

### **1. How Terraform Works**

Terraform is an Infrastructure as Code (IaC) tool that lets you define cloud infrastructure in code. It allows you to automate provisioning and manage the lifecycle of resources. Here’s the general workflow:

1. **Write Configuration**: Define your infrastructure in configuration files using HashiCorp Configuration Language (HCL).
2. **Initialize**: Set up Terraform in your project directory with `terraform init`.
3. **Plan**: Review changes Terraform will make without applying them using `terraform plan`.
4. **Apply**: Provision resources as specified with `terraform apply`.
5. **Manage State**: Terraform uses a state file to track resources and ensure changes are applied consistently.
6. **Destroy**: Remove all managed resources when they are no longer needed with `terraform destroy`.

Terraform manages resource dependencies automatically, meaning it will create resources in the order required, based on how they reference each other.

---

### **2. Terraform Configuration: AWS Architecture Overview**

In this example, we’ll build a basic AWS setup with the following resources:

1. **VPC (Virtual Private Cloud)**: A virtual network for isolated resources.
2. **Public Subnet**: A subnet within the VPC, configured to assign public IP addresses to resources.
3. **Internet Gateway (IGW)**: Provides internet access for resources in the public subnet.
4. **Route Table**: Defines routes for directing traffic, including internet-bound traffic.
5. **Security Group**: Sets firewall rules to allow inbound/outbound traffic for specified ports.
6. **EC2 Instance**: A virtual server in the public subnet, accessible over the internet.

---

### **3. Writing the Terraform Configuration**

Let’s look at each component of the Terraform configuration file. 

**Step 1: Provider Setup**

The provider block tells Terraform to use AWS as the cloud provider and sets the region.

```hcl
# Provider setup for AWS
provider "aws" {
  region = "eu-central-1"
}
```

**Step 2: Create a VPC**

A Virtual Private Cloud (VPC) is a virtual network in AWS. We specify the `cidr_block` to define the IP range.

```hcl
# VPC Creation
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Sharanya-VPC"
  }
}
```

**Step 3: Create a Public Subnet**

The public subnet is part of the VPC and has a unique CIDR block. Setting `map_public_ip_on_launch = true` enables automatic assignment of public IPs to resources in this subnet.

```hcl
# Public Subnet within the VPC
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "eu-central-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "Sharanya-PublicSubnet"
  }
}
```

**Step 4: Create an Internet Gateway**

An Internet Gateway (IGW) allows resources in the VPC to access the internet. This is essential for public subnets.

```hcl
# Internet Gateway for VPC
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "Sharanya-InternetGateway"
  }
}
```

**Step 5: Create a Route Table**

The route table defines the routes for network traffic, including routes to the internet.

```hcl
# Route Table for Public Subnet
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "Sharanya-PublicRouteTable"
  }
}
```

**Step 6: Add a Route to the Internet**

Here, we add a route to direct internet-bound traffic (0.0.0.0/0) to the Internet Gateway.

```hcl
# Default Route to the Internet
resource "aws_route" "internet_access" {
  route_table_id         = aws_route_table.public_rt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}
```

**Step 7: Associate the Route Table with the Subnet**

This associates the public route table with the public subnet, allowing instances in the subnet to use the specified routes.

```hcl
# Associate Route Table with Public Subnet
resource "aws_route_table_association" "public_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}
```

**Step 8: Create a Security Group**

The security group acts as a firewall, allowing specific traffic. Here, we allow SSH (port 22) and HTTP (port 80) traffic.

```hcl
# Security Group for SSH and HTTP
resource "aws_security_group" "my_sg" {
  vpc_id = aws_vpc.my_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Sharanya-SecurityGroup"
  }
}
```

**Step 9: Launch an EC2 Instance**

Finally, we create an EC2 instance, associating it with the public subnet and security group.

```hcl
# EC2 Instance within Public Subnet
resource "aws_instance" "my_ec2" {
  ami                    = "ami-0084a47cc718c111a" # Amazon Linux 2 AMI in eu-central-1
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public_subnet.id
  vpc_security_group_ids = [aws_security_group.my_sg.id]

  tags = {
    Name = "Sharanya-EC2Instance"
  }
}
```

**Output Values**

These output values make it easy to retrieve the VPC ID, subnet ID, and EC2 public IP after running Terraform.

```hcl
output "vpc_id" {
  value = aws_vpc.my_vpc.id
}

output "subnet_id" {
  value = aws_subnet.public_subnet.id
}

output "ec2_public_ip" {
  value = aws_instance.my_ec2.public_ip
}
```

---

### **4. Understanding Implicit Dependency**

Terraform automatically understands resource dependencies based on references. For example, the public subnet references `aws_vpc.my_vpc.id`, so Terraform knows the VPC must be created first. Similarly, the EC2 instance references `aws_subnet.public_subnet.id` and `aws_security_group.my_sg.id`, so Terraform will create the subnet and security group before the instance.

---

### **5. Essential Terraform Commands**

Here are some key commands you’ll use to manage this configuration:

- **`terraform init`**: Initializes the project directory for Terraform, downloading any required plugins.
- **`terraform fmt`**: Formats configuration files for readability.
- **`terraform validate`**: Validates syntax and structure before planning changes.
- **`terraform plan`**: Previews changes Terraform will make to the infrastructure.
- **`terraform apply`**: Applies the planned changes, provisioning the resources.
- **`terraform state list`**: Lists resources in the current state.
- **`terraform output`**: Shows values of output variables defined in the configuration.
- **`terraform graph`**: Generates a visual graph of resource dependencies.
- **`terraform destroy`**: Destroys all resources defined in the configuration.

---

### **Conclusion**

In this blog, we’ve covered the basics of Terraform, explored a simple AWS setup, and seen how implicit dependencies work. By using Terraform to manage resources, you can consistently provision and maintain your infrastructure, keeping configurations simple yet powerful.
