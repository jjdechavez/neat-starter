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
Resources (Blog AWS Resizing images with Amazon CloudFront and Lambda Edge)\[https://aws.amazon.com/blogs/networking-and-content-delivery/resizing-images-with-amazon-cloudfront-lambdaedge-aws-cdn-blog/]

Requirements:

1. S3 bucket (Public)
2. IAM Role for lambda edge
3. Two Lambda function for Viewer Request and Origin Response (Deploy on Lambda@Edge)
4. CloudFront Distribution 

Creating S3 Bucket

1. Go to your AWS Console and open s3.
2. Create a new Bucket, make sure the name of the bucket would be the same on CloudFront CNAME let say our bucket name is assets.jjdechavez.dev.
3. Make sure set AWS Region on us-east-1 because Lambda@Edge is only available on us-east-1.
4. On Object Ownership, select ACLs enabled, and the Object Ownership is Bucket owner preferred
5. On Block Public Access settings for this bucket, uncheck the **Block all public access** and check the **"I acknowledge that the current settings might result in this bucket and the objects within becoming public."**
6. Then create bucket.

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

     ```json
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
6. On Permission tab of s3 bucket, click Edit on Cross-origin resource sharing (CORS) and copy the code below

   ```json
   [
       {
           "AllowedHeaders": [
               "*"
           ],
           "AllowedMethods": [
               "PUT"
           ],
           "AllowedOrigins": [
               "*"
           ],
           "ExposeHeaders": [],
           "MaxAgeSeconds": 3000
       }
   ]
   ```
7. Click the save changes.

Create Lambda Edge for Origin Response and Viewer Request

1. First, we are going to create Origin Response lambda function. Go to the lambda and click Create Function.
2. Set function option to Author from scratch. Then enter the lambda name in our example we are going to name as uploadOriginResponse.
3. Set Runtime as Node.js 14.x because lambda edge only works on Node.js 14.
4. Set Architecture as x86_64 and click Create Function button.
5. Open our created lambda function named uploadOriginResponse. Copy the Origin response code. Here it checks and invokes the resize trigger if required.

   ```javascript
   'use strict';

   const http = require('http');
   const https = require('https');
   const querystring = require('querystring');

   const AWS = require('aws-sdk');
   const S3 = new AWS.S3({
     signatureVersion: 'v4',
   });
   const Sharp = require('sharp');

   // set the S3 and API GW endpoints
   const BUCKET = 'image-resize-${AWS::AccountId}-us-east-1';

   exports.handler = (event, context, callback) => {
     let response = event.Records[0].cf.response;

     console.log("Response status code :%s", response.status);

     //check if image is not present
     if (response.status == 404) {

       let request = event.Records[0].cf.request;
       let params = querystring.parse(request.querystring);

       // if there is no dimension attribute, just pass the response
       if (!params.d) {
         callback(null, response);
         return;
       }

       // read the dimension parameter value = width x height and split it by 'x'
       let dimensionMatch = params.d.split("x");

       // read the required path. Ex: uri /images/100x100/webp/image.jpg
       let path = request.uri;

       // read the S3 key from the path variable.
       // Ex: path variable /images/100x100/webp/image.jpg
       let key = path.substring(1);

       // parse the prefix, width, height and image name
       // Ex: key=images/200x200/webp/image.jpg
       let prefix, originalKey, match, width, height, requiredFormat, imageName;
       let startIndex;

       try {
         match = key.match(/(.*)\/(\d+)x(\d+)\/(.*)\/(.*)/);
         prefix = match[1];
         width = parseInt(match[2], 10);
         height = parseInt(match[3], 10);

         // correction for jpg required for 'Sharp'
         requiredFormat = match[4] == "jpg" ? "jpeg" : match[4];
         imageName = match[5];
         originalKey = prefix + "/" + imageName;
       }
       catch (err) {
         // no prefix exist for image..
         console.log("no prefix present..");
         match = key.match(/(\d+)x(\d+)\/(.*)\/(.*)/);
         width = parseInt(match[1], 10);
         height = parseInt(match[2], 10);

         // correction for jpg required for 'Sharp'
         requiredFormat = match[3] == "jpg" ? "jpeg" : match[3]; 
         imageName = match[4];
         originalKey = imageName;
       }

       // get the source image file
       S3.getObject({ Bucket: BUCKET, Key: originalKey }).promise()
         // perform the resize operation
         .then(data => Sharp(data.Body)
           .resize(width, height)
           .toFormat(requiredFormat)
           .toBuffer()
         )
         .then(buffer => {
           // save the resized object to S3 bucket with appropriate object key.
           S3.putObject({
               Body: buffer,
               Bucket: BUCKET,
               ContentType: 'image/' + requiredFormat,
               CacheControl: 'max-age=31536000',
               Key: key,
               StorageClass: 'STANDARD'
           }).promise()
           // even if there is exception in saving the object we send back the generated
           // image back to viewer below
           .catch(() => { console.log("Exception while writing resized image to bucket")});

           // generate a binary response with resized image
           response.status = 200;
           response.body = buffer.toString('base64');
           response.bodyEncoding = 'base64';
           response.headers['content-type'] = [{ key: 'Content-Type', value: 'image/' + requiredFormat }];
           callback(null, response);
         })
       .catch( err => {
         console.log("Exception while reading source image :%j",err);
       });
     } // end of if block checking response statusCode
     else {
       // allow the response to pass through
       callback(null, response);
     }
   };
   ```
6. We need to zip this code with package.json and node_modules and upload to our lambda function.
7. And for our Viewer Request lambda function kindly follow the steps ^ on how creating lambda function. Copy the code. Here it manipulates our request uri.

   ```javascript
   'use strict';

   const querystring = require('querystring');

   // defines the allowed dimensions, default dimensions and how much variance from allowed
   // dimension is allowed.

   const variables = {
           allowedDimension : [ {w:100,h:100}, {w:200,h:200}, {w:300,h:300}, {w:400,h:400} ],
           defaultDimension : {w:200,h:200},
           variance: 20,
           webpExtension: 'webp'
     };

   exports.handler = (event, context, callback) => {
       const request = event.Records[0].cf.request;
       const headers = request.headers;

       // parse the querystrings key-value pairs. In our case it would be d=100x100
       const params = querystring.parse(request.querystring);

       // fetch the uri of original image
       let fwdUri = request.uri;

       // if there is no dimension attribute, just pass the request
       if(!params.d){
           callback(null, request);
           return;
       }
       // read the dimension parameter value = width x height and split it by 'x'
       const dimensionMatch = params.d.split("x");

       // set the width and height parameters
       let width = dimensionMatch[0];
       let height = dimensionMatch[1];

       // parse the prefix, image name and extension from the uri.
       // In our case /images/image.jpg

       const match = fwdUri.match(/(.*)\/(.*)\.(.*)/);

       let prefix = match[1];
       let imageName = match[2];
       let extension = match[3];

       // define variable to be set to true if requested dimension is allowed.
       let matchFound = false;

       // calculate the acceptable variance. If image dimension is 105 and is within acceptable
       // range, then in our case, the dimension would be corrected to 100.
       let variancePercent = (variables.variance/100);

       for (let dimension of variables.allowedDimension) {
           let minWidth = dimension.w - (dimension.w * variancePercent);
           let maxWidth = dimension.w + (dimension.w * variancePercent);
           if(width >= minWidth && width <= maxWidth){
               width = dimension.w;
               height = dimension.h;
               matchFound = true;
               break;
           }
       }
       // if no match is found from allowed dimension with variance then set to default
       //dimensions.
       if(!matchFound){
           width = variables.defaultDimension.w;
           height = variables.defaultDimension.h;
       }

       // read the accept header to determine if webP is supported.
       let accept = headers['accept']?headers['accept'][0].value:"";

       let url = [];
       // build the new uri to be forwarded upstream
       url.push(prefix);
       url.push(width+"x"+height);
     
       // check support for webp
       if (accept.includes(variables.webpExtension)) {
           url.push(variables.webpExtension);
       }
       else{
           url.push(extension);
       }
       url.push(imageName+"."+extension);

       fwdUri = url.join("/");

       // final modified url is of format /images/200x200/webp/image.jpg
       request.uri = fwdUri;
       callback(null, request);
   };
   ```
8. Zip the viewer request code, for our example we are going to name this function as uploadViewerRequest and upload the zip on the lambda function.

Creating CloudFront Distribution

1. Go to the AWS CloudFront, and click Create Distribution.
2. On Origin, select the s3 that we created, in our example is assets.jjdechavez.dev.s3.us-east-1.amazonaws.com
3. On Default cache behavior section, on Viewer set as Redirect HTTP to HTTPS.
4. On Cache policy, we are going to create a custom cache policy by click create policy

   * Set a name, in our example we are going to name as CF-Cache-Lambda-Edge.
   * On Cache setting, select Include specified query string and add on allow **d** query string.
5. Click Create policy.
6. On Cache Policy, select our custom policy named CF-Cache-Lambda-Edge.
7. On Function associations, Viewer Request Function type select Lambda@Edge and get the lambda Viewer Request ARN and paste on Function ARN field. The same goes to the Origin Response.
8. Click Create Distribution.

Deploy our Origin Response and Viewer Request Lambda Function

1. Go to the AWS Lambda and open Origin Response name as uploadOriginResponse.
2. On Action, click Deploy to Lambda@Edge. Select Configure new CloudFront trigger option.
3. Get the CloudFront Distribution ARN and paste it.
4. For our CloudFront event, select Origin Response and lastly, check Confirm deploy to Lambda@Edge.
5. Wait deployment to be finish to check the status, go to the CloudFront distributions. On the list, check the Column Last Modified if it's deploying or already done.
6. Go to the AWS Lambda and open Viewer Request name as uploadViewerRequest.
7. On Action, click Deploy to Lambda@Edge. Select Configure new CloudFront trigger option.
8. Get the CloudFront Distribution ARN and paste it.
9. For our CloudFront event, select Viewer Request and lastly, check Confirm deploy to Lambda@Edge.
10. Go back to step 5.



Testing our implementation

1. Upload a high-res image file (let’s call it image.jpg) into ‘images’ folder on the origin bucket created.
2. Open your favorite browser and navigate to `https://{cloudfront-domain}/images/image.jpg?d=100x100` where

* cloudfront-domain – is the CloudFront domain name of the distribution created using the CloudFormation template above.
* 100×100 – is the desired width and height. Change the dimension by altering value of query parameter ‘d’ to 200×200 or 300×300.

You should see the corresponding resized image.