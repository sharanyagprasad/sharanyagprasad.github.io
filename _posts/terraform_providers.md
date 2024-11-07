---
title: Understanding Terraform Providers and the `providers` Meta-Argument
description: To know how provider meta argument is used within the resources block 
author: sharanya
date: 2024-11-07 17:05:00 +0800
categories: [Getting Started, Blog]
tags: [terraform, IaC, AWS]
# image:
#   path: /assets/img/make_a_blog_site/kaitlyn-baker-unsplash.jpg
#   alt: Photo by Kaitlyn Baker on Unsplash

---


In Terraform, providers are essential for provisioning and managing infrastructure across multiple platforms. They bridge Terraform with APIs from various platforms like AWS, Google Cloud, and others, allowing users to define resources such as compute instances, storage buckets, and databases.

This article provides a complete overview of Terraform providers, explains the `providers` meta-argument, and demonstrates practical examples to help you get started with multi-region and multi-account setups.

---

### What is a Provider in Terraform?

A **provider** in Terraform acts as a plugin that enables Terraform to communicate with APIs of various platforms like AWS, Azure, Google Cloud, etc. Each provider allows Terraform to create, manage, and interact with resources from that specific platform. In a typical Terraform setup, you define providers in your configuration file to specify which infrastructure platform you intend to use.

For example, if you're working with AWS, a provider configuration may look like this:

```hcl
provider "aws" {
  region = "us-west-2"
}
```

This example specifies AWS as the provider and sets the region to us-west-2. When defining resources like EC2 instances or S3 buckets, Terraform will use this configuration to communicate with the AWS API.

### The providers Meta-Argument
In addition to setting up providers, Terraform offers a providers meta-argument to manage resources across multiple configurations of the same provider. This is useful when you need to:

- Deploy resources in multiple regions within the same account (e.g., us-east-1 and eu-central-1).
- Manage resources across different accounts (e.g., production and development accounts).
- Isolate specific resources to particular provider configurations for better control and security.

The providers meta-argument allows you to specify which provider configuration a resource should use, enabling seamless multi-region or multi-account setups within the same configuration file.

## Example 1: Multi-Region Setup in AWS Using the providers Meta-Argument
Let’s start with a simple example of deploying EC2 instances in multiple AWS regions (eu-central-1 and us-east-1). This example also includes an output block to show the region where each instance is deployed.

```hcl
provider "aws" {
  region = "eu-central-1"
}

provider "aws" {
  region = "us-east-1"
  alias  = "us-east"
}

resource "aws_instance" "us-east" {
  instance_type = "t2.micro"
  provider      = aws.us-east
  ami           = "ami-0866a3c8686eaeeba"
  tags = {
    Name = "East-Instance"
  }
}

resource "aws_instance" "eu-central" {
  instance_type = "t2.nano"
  ami           = "ami-0084a47cc718c111a"
  tags = {
    Name = "eucentral-Instance"
  }
}

output "instance_regions" {
  description = "Regions of the EC2 instances"
  value = {
    "us-east-instance" = aws_instance.us-east.provider.region
    "eu-central-instance" = aws_instance.eu-central.provider.region
  }
}
```
### Explanation
- Multiple Providers: We define two AWS providers:

    - The default AWS provider for eu-central-1.
    - An additional AWS provider with an alias us-east for us-east-1.
- Using the providers Meta-Argument: The aws_instance.us-east resource explicitly specifies the aws.us-east provider, ensuring it’s created in us-east-1, while aws_instance.eu-central uses the default provider (eu-central-1).

- Output Block: The output block instance_regions provides the region of each EC2 instance, demonstrating the usage of multiple provider configurations within the same configuration file.


## Example 2: Multi-Account Setup Using the providers Meta-Argument
In some setups, organizations use separate AWS accounts for development and production environments. Here’s how we can define two separate provider configurations with different credentials and manage resources in each account.

```hcl
provider "aws" {
  region     = "us-west-2"
  alias      = "production"
  access_key = "PROD_ACCESS_KEY"
  secret_key = "PROD_SECRET_KEY"
}

provider "aws" {
  region     = "us-west-2"
  alias      = "development"
  access_key = "DEV_ACCESS_KEY"
  secret_key = "DEV_SECRET_KEY"
}

resource "aws_s3_bucket" "prod_bucket" {
  provider = aws.production
  bucket   = "my-prod-bucket"

  tags = {
    Name = "Production Bucket"
    Env  = "Production"
  }
}

resource "aws_s3_bucket_acl" "prod_bucket_acl" {
  provider = aws.production
  bucket   = aws_s3_bucket.prod_bucket.id
  acl      = "private"
}

resource "aws_s3_bucket" "dev_bucket" {
  provider = aws.development
  bucket   = "my-dev-bucket"

  tags = {
    Name = "Development Bucket"
    Env  = "Development"
  }
}

resource "aws_s3_bucket_acl" "dev_bucket_acl" {
  provider = aws.development
  bucket   = aws_s3_bucket.dev_bucket.id
  acl      = "private"
}
```

### Explanation
- Separate Provider Configurations for Each Account:

    - aws.production uses access credentials for the production account.
    - aws.development uses access credentials for the development account.
- Resource Isolation by Account: The aws_s3_bucket resources are created in separate accounts based on the provider specified in each resource:

    - aws_s3_bucket.prod_bucket is created in the production account.
    - aws_s3_bucket.dev_bucket is created in the development account.
- Separate ACL Resources: Since the acl attribute is now managed via a separate resource, aws_s3_bucket_acl is used to set the access level (private) for each bucket.

## Key Takeaways
- Flexibility with Providers: Terraform providers enable seamless integration with various platforms. Defining multiple providers in a configuration allows greater control over where resources are deployed.
- Multi-Region and Multi-Account Management: The providers meta-argument is especially useful in multi-region and multi-account setups, enabling you to specify which provider configuration should be used per resource.
- Separation of Concerns: The separate aws_s3_bucket_acl resource offers improved management for access control and avoids deprecated warnings in newer Terraform versions.

By leveraging Terraform providers and the providers meta-argument effectively, you can build and manage infrastructure across regions, accounts, and environments—all from a single configuration file.


