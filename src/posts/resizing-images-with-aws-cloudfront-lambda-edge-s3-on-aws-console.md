---
title: Resizing Images with AWS CloudFront & Lambda@Edge & S3 on AWS Console
description: Guide for creating resizing images with AWS CloudFront &
  Lambda@Edge & S3 on AWS Console
author: John Jerald De Chavez
date: 2022-07-26T01:11:00.706Z
tags:
  - aws
  - lambda@edge
  - s3
---
Resources (Blog AWS Resizing images with Amazon Cloudfront and Lambda Edge)\[https://aws.amazon.com/blogs/networking-and-content-delivery/resizing-images-with-amazon-cloudfront-lambdaedge-aws-cdn-blog/]

Requirements:

1. S3 bucket (Public)
2. IAM Role for lambda edge
3. Two Lambda function for Viewer Request and Origin Response (Deploy on Lambda@Edge)
4. CloudFront Distribution 

Creating S3 Bucket

1. Go to your AWS Console and open s3.
2. Create a new Bucket, make sure the name of the bucket would be the same on CloudFront CNAME let say our bucket name is assets.jjdechavez.dev.
3. Make sure set AWS Region on us-east-1 because Lambda@Edge is only available on us-east-1.
4. On Block Public Access settings for this bucket, uncheck the **Block all public access** and check the **"I acknowledge that the current settings might result in this bucket and the objects within becoming public."**
5. Then create bucket.

Create IAM Role for Lambda edge

1. Go to AWS IAM and open Roles, and create Role.
2. Set Trusted entity type as AWS Service and Common use cases as Lambda, then click next button.
3. Select policy name [AWSLambdaBasicExecutionRole](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::625472463376:policy/service-role/AWSLambdaBasicExecutionRole-10c3a2d5-a0fd-4511-ab2c-2ebdc7f6d24d) , [AWSLambdaEdgeExecutionRole](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::625472463376:policy/service-role/AWSLambdaEdgeExecutionRole-8121fb11-5c37-44eb-8c02-dc3af3e556b8) and optional S3 policy action (s3:PutObject, s3:GetObject). Then click next button.
4. Add role name for this example we are going to name as Lambda-Edge-Role.
5. Click the Create Role button.

Configure S3 Bucket

1. Open the created s3 bucket in our example, our bucket name is assets.jjdechavez.dev
2. Go to the properties tab, edit **Static website hosting**  

   * Enable static web hosting
   * Set Hosting Type to Host a static website
   * Set Index document to index.html on input field
   * Save the changes.
3. Go to the permissions tab of the s3 bucket and edit Bucket Policy.
4. Copy and paste the policy, what we need is ARN of our s3 bucket and ARN of our IAM role that we created.

   * Please attach your ARN s3 and IAM Role, code sample show below.

     ```
     {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "Statement1",
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::assets.jjdechavez.dev/*"
          },
          {
              "Sid": "Statement2",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::9423437289:role/service-role/Lambda-Edge-Role"
              },
              "Action": "s3:PutObject",
              "Resource": "arn:aws:s3:::assets.jjdechavez.dev/*"
          },
          {
              "Sid": "Statement3",
              "Effect": "Allow",
              "Principal": {
                  "AWS": "arn:aws:iam::9423437289:role/service-role/Lambda-Edge-Role"
              },
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::assets.jjdechavez.dev/*"
          }
      ]
     }
     ```
5. Click Save changes button.