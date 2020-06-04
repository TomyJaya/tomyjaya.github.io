---
layout:     post
title:      "Trying AWS Lambda"
subtitle:   "Documented Steps"
date:       2017-05-14 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- aws
- lambda
- serverless
---

# Pre-amble

Serverless, or more specifically in the AWS world, Lambda, seems to be the buzzword recently. So, I decided to give it a try. Getting an appropriate use-case is the difficult one. A simple Hello World tutorial wouldn't give you enough exposure. Hence, after much thoughts, I decided to try to "Lambda"-dize (convert existing backendful service to a Lambda function) a function to send email notification upon contact form's submission from my static website. Not to complex but not to simple either.

# Pre-requisites
1. Familiarity with the AWS Console
2. Basic knowledge of node, npm, and JavaScript as we'll be using the Node API
3. Basic knowledge of IAM (e.g. Role/ Policy), S3, and SES

# Setting Up Your Lambda Function

Follow the steps here: [https://github.com/eleven41/aws-lambda-send-ses-email](https://github.com/eleven41/aws-lambda-send-ses-email)

Some helpful pointers/ resources: 
1. [Creating a New Policy Using Policy Editor](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-editor)
2. When creating the deployment package, go to the SendSesEmail folder of the downloaded zip and type `npm install` in the Terminal
3. When zipping the deployment package, make sure you zip the content files instead of the SendSesEmail directory. 
4. Follow the instructions [here](http://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started.html#getting-started-new-lambda) to create a lambda function. But instead of writing the code in the web editor, opt to upload your zip file. 

# Setting up 

The Lambda function needs to be exposed as a RESTful service. Here's where API Gateway comes into the picture. Steps can be found below:

[Build an API to Expose a Lambda Function](http://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started.html)


# Caveats (README!!!)

1. SES is only available in selected regions. At first, I tried with ap-southeast-1 region Singapore and encountered region not available issue. At the end, I decided to host everything in us-east-1. 

2. Make sure to run `npm install` to download all the dependencies before zipping your deployment package. You will encounter 'markup-js' not found error if you don't. 

3. Make sure you zip the content files instead of the directory when creating the deployment package. 

4. Somehow sending to a Yahoo mail server doesn't work with SES sandbox that well. My GMail account has no problem though. 
