# AWS-Immersion-Day for CNF/K8s engineers (dev/test) in Telco / Graviton2 - arm64

In this lab we deploy AWS EKS (Elastic Kubernetes Service) and related constructs and AWS Graviton2 (arm64) based worker nodes supporting Multus.

See AWS EKS Multus Setup Guide for more info about Multus and EKS: https://github.com/aws-samples/eks-install-guide-for-multus 

## AWS Region
* ONLY use 'us-west-2' (Oregon) Region during these labs.

## Download GitHub as zip (for CloudFormation Template and Lambda.zip file)
For the download, please select "Code" on the top left corner of menu, and then click green "Code" menu and then select "Download Zip" file.
Alternate option is to clone this repository to your workstation.

## Log in to AWS Event Engine
* https://dashboard.eventengine.run/dashboard
* Type in Event Hash that is shared by your instructor
* Please use OTP authentication with your email
  ![Otp](Lab1/images/otp.png)
1. Click "Set Team Name"
    * Use your name as team name  
    ![Dashboard](Lab1/images/dashboard-aws.png)
2. Click "SSH key" 
    * Download **ee-default-keypair** to your PC - also copy key material to notepad
3. Click "AWS Console"
    * Copy credentials (export AWS_DEFAULT_REGION=..) 
    * Click "Open AWS Console"

## What we will learn today? 
* **[Lab1](https://github.com/TheHannuAWS/AWS-Immersion-Day/tree/main/Lab1)**: EKS environment creation (VPC, Subnets, EKS cluster, Multus-ready worker node) for CNF deployment

* **[Lab2](https://github.com/TheHannuAWS/AWS-Immersion-Day/tree/main/Lab2)**: Create dummy Pod Application (CNF) using Multus - build image and deploy from Amazon ECR
