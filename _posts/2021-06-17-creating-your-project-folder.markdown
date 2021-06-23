---
layout: post
title:  "Creating Your First Project Folder"
date:   2021-06-17 23:17:45 -0400
categories: aws terraform iac packer hashicorp
---
As mentioned in my previous post, getting started with Terraform is very easy! My favorite aspect of terraform is the easy of use and power it affords. I won't cover installing terraform here, check out the [Install Terraform][terraform-install] video guide from Hashicorp for a better description that I could provide.

One of the first considerations you'll have to make when working with terraform is to decide how you intend to store the credentials terraform will be using to provision infrastructure on your behalf. While terraform does support hard coding `access_key` and `secret_key`, this is ill-advised for many reasons. The most obvious being accidentally making those credentials visible in a code repository.

The two best methods to pass in credentials are: environment variables and IAM credentials

Wait a minute, IAM credentials? I don't even have an AWS account yet. How am I supposed to 





[terraform-install]: https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started
[terraform-aws-provider]: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
