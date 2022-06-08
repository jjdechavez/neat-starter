---
title: "AWS: Update API params old to new params"
description: AWS API Gateway update params
author: John Jerald E. De Chavez
date: 2022-06-08T08:18:13.398Z
tags:
  - aws
  - api
---
How to update api params old to new params

1. Open AWS Console
2. Go API Gateway > Custom Domains > Mappings > Unmap the api > Save
3. Go to CloudFormation > Find the api and select it to remove.

If deleting api on cloudformation is faild or status "Delete Failed"

1. Go to Api's resources tab > Search s3 bucket > Remove the serverless object
2. After removing, go back to Cloud Formation then remove the api