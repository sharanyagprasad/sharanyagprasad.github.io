---
title: Power of Math: An End-to-End Web Application with AWS
description: This project demonstrates how to build a simple yet powerful web application using AWS services. The application, called "The Power of Math", allows users to input two numbers (a base and an exponent) and calculates the result of raising the base to the power of the exponent. The result is displayed to the user and stored in a DynamoDB table for future reference.
author: sharanya
date: 2025-02-27 14:55:00 +0800
categories: [AWS]
tags: [AWS, DynamoDB, API Gateway, Lambda, AWS Amplify, IAM policy, IAM,  ]
image:
  path: /assets/img/aws/power_of_math.png
  alt: AWS architecture using Amplify, API Gateway, Lambda and DynamoDB

---

# Power of Math: An End-to-End Web Application with AWS

This project demonstrates how to build a simple yet powerful web application using AWS services. The application, called **"The Power of Math"**, allows users to input two numbers (a base and an exponent) and calculates the result of raising the base to the power of the exponent. The result is displayed to the user and stored in a DynamoDB table for future reference.

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Step 1: Creating and Hosting the Web Page with AWS Amplify](#step-1-creating-and-hosting-the-web-page-with-aws-amplify)
5. [Step 2: Creating a Lambda Function for Math Calculations](#step-2-creating-a-lambda-function-for-math-calculations)
6. [Step 3: Creating an API Gateway to Invoke the Lambda Function](#step-3-creating-an-api-gateway-to-invoke-the-lambda-function)
7. [Step 4: Storing Results in DynamoDB](#step-4-storing-results-in-dynamodb)
8. [Step 5: Connecting the Frontend to the Backend](#step-5-connecting-the-frontend-to-the-backend)
9. [Conclusion](#conclusion)

---

## Introduction

The goal of this project is to build a web application that performs a simple mathematical calculation and stores the result in a database. The application is built using AWS services, which handle everything from hosting the web page to performing the calculation and storing the result.

---

## Architecture Overview

The application consists of the following components:

1. **AWS Amplify**: Hosts the static web page.
2. **API Gateway**: Provides a REST API endpoint to trigger the Lambda function.
3. **Lambda**: Performs the mathematical calculation.
4. **DynamoDB**: Stores the result of the calculation.
5. **IAM**: Manages permissions for the Lambda function to write to DynamoDB.

---

## Prerequisites

Before starting, ensure you have the following:

- An **AWS account**.
- A **text editor** (e.g., Notepad++, VS Code).
- Basic knowledge of **HTML**, **JavaScript**, and **Python**.
- Basic familiarity with AWS services.

---

## Step 1: Creating and Hosting the Web Page with AWS Amplify

### 1.1 Create the HTML File

Create a file named `index.html` and paste the following code:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
    <style>
        /* Add your CSS styling here */
    </style>
    <script>
        // JavaScript to call the API Gateway endpoint
        var callAPI = (base, exponent) => {
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var raw = JSON.stringify({ "base": base, "exponent": exponent });
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                body: raw,
                redirect: 'follow'
            };
            fetch("YOUR_API_GATEWAY_ENDPOINT", requestOptions)
                .then(response => response.text())
                .then(result => alert(JSON.parse(result).body))
                .catch(error => console.log('error', error));
        }
    </script>
</head>
<body>
    <h1>TO THE POWER OF MATH!</h1>
    <form>
        <label>Base number:</label>
        <input type="text" id="base">
        <label>...to the power of:</label>
        <input type="text" id="exponent">
        <button type="button" onclick="callAPI(document.getElementById('base').value, document.getElementById('exponent').value)">CALCULATE</button>
    </form>
</body>
</html>

```


### 1.2 Host the Web Page with AWS Amplify
1. Zip the index.html file.

2. Go to AWS Amplify in the AWS Management Console.

3. Click Host web app and upload the ZIP file.

4. Deploy the application and note the URL provided by Amplify.

---


## Step 2: Creating a Lambda Function for Math Calculations
### 2.1 Create the Lambda Function
1. Go to AWS Lambda in the AWS Management Console.

2. Click Create function.

3. Name your function (e.g., PowerOfMathFunction).

4. Select Python 3.x as the runtime.

5. Paste the following code:

```python
import json
import math
import boto3
from time import gmtime, strftime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('PowerOfMathDatabase')
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime': now
        })
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }

```

### 2.2 Test the Lambda Function
1. Configure a test event with the following JSON:

```python
{
  "base": 2,
  "exponent": 3
}

```
2. Run the test to ensure the function works.

---

## Step 3: Creating an API Gateway to Invoke the Lambda Function
### 3.1 Create the API Gateway
1. Go to API Gateway in the AWS Management Console.

2. Create a new REST API.

3. Name your API (e.g., PowerOfMathAPI).

### 3.2 Create a POST Method
1. Create a new resource and method (POST).

2. Integrate the method with your Lambda function.

### 3.3 Deploy the API
1. Deploy the API to a new stage (e.g., dev).

3. Note the Invoke URL.

--

## Step 4: Storing Results in DynamoDB

### 4.1 Create a DynamoDB Table
1. Go to DynamoDB in the AWS Management Console.

2. Create a table named PowerOfMathDatabase with a primary key ID (String).

### 4.2 Update the Lambda Function
Ensure the Lambda function code references the correct DynamoDB table.

### 4.3 Add IAM Permissions
Attach the following policy to your Lambda function's IAM role:

```python
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query",
        "dynamodb:UpdateItem"
      ],
      "Resource": "YOUR-TABLE-ARN"
    }
  ]
}
```
---

## Step 5: Connecting the Frontend to the Backend
### 5.1 Update the index.html File
1. Replace `YOUR_API_GATEWAY_ENDPOINT` in the `index.html` file with the Invoke URL from API Gateway.

### 5.2 Redeploy the Web Page
1. Zip the updated index.html file.

2. Redeploy it in AWS Amplify.

--- 

# Conclusion
You have successfully built an end-to-end web application using AWS services. This project is a great addition to your portfolio and demonstrates your ability to work with AWS Amplify, API Gateway, Lambda, IAM, and DynamoDB.