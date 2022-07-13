---
title: AWS CLI Cheatsheet
description: Common AWS cli commands.
author: John Jerald De Chavez
date: 2022-07-13T02:31:21.483Z
tags:
  - aws
  - cli
---
Getting tired opening new tab for commands. So, I decided to collect the common commands on AWS CLI.

**[List users](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/list-users.html)**\
`aws cognito-idp list-users --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id`

With filter
`aws cognito-idp list-users --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id --filter 'cognito:user_status = "FORCE_CHANGE_PASSWORD"'`

**[Get user](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/admin-get-user.html)**\
`aws cognito-idp admin-get-user --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id --username johndoe@mail.com`

**[Resend temporary password](https://docs.aws.amazon.com/cli/latest/reference/cognito-idp/admin-create-user.html)**\
`aws cognito-idp admin-create-user --profile your-aws-profile --region us-east-1 --user-pool-id us-east-1_your-pool-id --username johndoe@mail.com --message-action RESEND`