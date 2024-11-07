---
title: Mastering Terraform Variables- A Complete Guide with Examples
description: To know how terraform variables, data types and need for .tfvars file
author: sharanya
date: 2024-11-07 22:17:00 +0800
categories: [Terraform]
tags: [terraform, IaC, AWS, variables, data types]
image:
  path: /assets/img/Terraform/Terraform.png
  alt: Terraform provider argument

---

# Mastering Terraform Variables: A Complete Guide with Examples

---

## Introduction

Terraform variables are a powerful feature in HashiCorp’s Terraform, allowing users to make infrastructure code dynamic, reusable, and manageable. By defining variables in configuration files, Terraform makes it easy to customize deployments across different environments without altering the actual code. In this guide, we’ll dive into Terraform variables, their syntax, data types, and the different ways to set and pass variable values. Additionally, we’ll look at the best practices for managing variables in Terraform.

---

## Table of Contents
1. [What Are Terraform Variables?](#what-are-terraform-variables)
2. [Types of Terraform Variables](#types-of-terraform-variables)
3. [Syntax of the `variable` Block](#syntax-of-the-variable-block)
4. [Example Terraform Configuration with Variables](#example-terraform-configuration-with-variables)
5. [Passing Variables Using Command-Line Flags](#passing-variables-using-command-line-flags)
6. [Handling Unset Variables](#handling-unset-variables)
7. [Using `.tfvars` Files for Variable Values](#using-tfvars-files-for-variable-values)
8. [Conclusion](#conclusion)

---

## What Are Terraform Variables?

Terraform variables allow for dynamic input in configuration files, enhancing flexibility and reusability. With variables, you can define values outside the main configuration files, making it easier to deploy consistent infrastructure across various environments.

### **Uses of Terraform Variables**
- **Reusability**: Variables enable infrastructure code to be reused across environments.
- **Maintainability**: By defining values in a centralized location, it becomes easier to manage configurations.
- **Consistency**: Using variables ensures that common values, like instance type or region, remain consistent throughout your code.

### **Pros and Cons of Using Terraform Variables**

| Pros | Cons |
|------|------|
| Simplifies management of configuration | Can become complex if overused |
| Increases code reusability | Can lead to errors if not set properly |
| Provides flexibility for different environments | Managing sensitive data through variables requires extra security |

---

## Types of Terraform Variables

Terraform supports various data types to provide flexibility when defining variables:

1. **String**: A single line of text, like `"t2.micro"`.
2. **Number**: A numeric value, like `1` or `0.5`.
3. **Boolean**: A true or false value, such as `true` or `false`.
4. **List**: A sequence of values, often used for IP addresses or port numbers, like `[22, 80, 443]`.
5. **Map**: Key-value pairs for more structured data, like `{ "Name" = "Example" }`.

---

## Syntax of the `variable` Block

The `variable` block in Terraform defines a variable’s type, description, default value, and more.

### **Mandatory Fields**
- **`variable "name"`**: Declares the name of the variable block.

### **Optional Fields**
- **`type`**: Specifies the data type (string, number, list, etc.).
- **`default`**: Sets a default value; if not provided, the variable will be required unless specified elsewhere.
- **`description`**: Provides context about the variable for readability.
  
Example:
```hcl
variable "instance_type" {
  type        = string
  description = "The type of EC2 instance to launch"
  default     = "t2.micro"
}
```

---

## Example Terraform Configuration with Variables

Here’s a sample configuration file demonstrating variables of different data types, along with an EC2 instance and security group:

```hcl
# variables.tf
variable "instance_type" {
  type        = string
  description = "The type of EC2 instance to launch"
  default     = "t2.micro"
}

variable "instance_count" {
  type        = number
  description = "The number of EC2 instances to create"
  default     = 1
}

variable "allowed_ports" {
  type        = list(number)
  description = "List of allowed ports for security group ingress rules"
  default     = [22, 80, 443]
}

variable "tags" {
  type        = map(string)
  description = "Map of tags to assign to the resources"
  default     = {
    "Environment" = "Dev"
    "Owner"       = "Student"
  }
}

# main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_security_group" "example_sg" {
  name        = "example-sg"
  description = "Example security group"
  
  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  tags = var.tags
}

resource "aws_instance" "example_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  count         = var.instance_count
  vpc_security_group_ids = [aws_security_group.example_sg.id]

  tags = var.tags
}
```

---

## Passing Variables Using Command-Line Flags

When running Terraform commands, you can pass variables directly through flags:

```sh
terraform apply -var="instance_type=t2.medium" -var="instance_count=2"
```

This approach is convenient for quickly testing configurations with different values without modifying the code.

---

## Handling Unset Variables

If a variable lacks a default value and is not set in a `.tfvars` file or through the command line, Terraform will prompt for the value at runtime:

```sh
var.instance_type
  Enter a value:
```

This feature allows flexibility but can slow down automation workflows if too many inputs are required.

---

## Using `.tfvars` Files for Variable Values

A `.tfvars` file is an external file containing variable values, making it easier to manage configuration changes and to keep your code organized.

### **Example `.tfvars` File (e.g., `terraform.tfvars`):**
```hcl
instance_type = "t2.small"
instance_count = 2
allowed_ports = [22, 8080]
tags = {
  Environment = "Production"
  Owner       = "Admin"
}
```

To use a specific `.tfvars` file:
```sh
terraform apply -var-file="production.tfvars"
```

### **Advantages of Using `.tfvars` Files**
- Simplifies passing multiple variables for different environments (e.g., development, staging, production).
- Keeps sensitive information like passwords or keys in a separate file (but remember to secure it).
- Reduces the chance of human error by centralizing variable management.

---

## Conclusion

Variables in Terraform enhance the flexibility, reusability, and manageability of your code. By leveraging variable data types, `.tfvars` files, and command-line flags, you can quickly configure infrastructure across environments and adapt configurations with ease. Keeping variable values separate from code encourages better organization and allows for more dynamic deployments. By mastering variables, you’ll make your Terraform configurations more powerful, secure, and efficient.
