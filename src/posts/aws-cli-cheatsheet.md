---
title: AWS CLI Cheatsheet
description: Common AWS cli commands.
author: John Jerald De Chavez
date: 2022-07-13T02:31:21.483Z
tags:
  - aws
  - cli
---
## Cheatsheet

**Get user**\
`aws cognito-idp admin-get-user --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id --username johndoe@mail.com`

**Resend temporary password**\
`aws cognito-idp admin-create-user --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id --username johndoe@mail.com --message-action RESEND`