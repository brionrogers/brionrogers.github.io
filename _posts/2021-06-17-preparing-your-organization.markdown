---
layout: post
title:  "Preparing Your Organization"
date:   2021-06-17 23:17:45 -0400
categories: aws terraform iac packer hashicorp
---
Before we can begin using terraform, we will need to deal with the chicken-egg problem of managing infrastructure in a fresh environment or project.

What chicken-egg problem you may ask? Try as we may, the reality is that there are some resources which will need be manually created before they can be administered via terraform. An example of that are AWS accounts. Though we have the `aws_organizations_account` resource which can provision accounts for us, we need to manually create the initial account from which we will administer the others in our organization.

AWS has a very good [tutorial][aws-new-account] for creating an account.

With our initial AWS account created, we need to take a pause to strategize a little bit. The account we just created has a real purpose and that purpose is to not get bogged down with half-configurations and development resources. A real effort should be made to keep this account clean and free of unnecessary resources. The reason we want to make this effort is because we do not want to inadvertently introduce a dependency where one may not need to exist. This however is where we arrive at one of the first pitfalls you can encounter with AWS. The actual topology of your AWS account structure will largely depend on the constraints imposed by organizational, governance, functional, and non-functional requirements. What works for someone may, in fact, be an absolute garbage arrangement for you and your organization.

For example, in this series I will describe an AWS organization with three accounts: the parent account and then production and non-production accounts. For my current job, this arrangement is more than sufficient. It provides isolation between the production and non-production accounts while also being relatively small in scale so that a relatively small team can administer it. However, larger organizations may have an AWS account per environment, they could have AWS accounts dedicated per team, separate accounts per specific AWS resource. At the end of the day, the specific layout you choose should balance the capability of your team to administer an environment (having two people to administer dozens of AWS accounts with hundreds of EC2 instances per account is inviting trouble) and the constraints the environment they administer face.

Pitfall: the topology of your terraform project should be thoughtful and is specific to your use case

Let's first create three additional resources manually:
1. S3 bucket for our remote state files
2. A DynamoDB table to implement locking of state files
3. An administative IAM role from which we can make changes to our root organizational account but also assume rules in our non-production/production accounts in order to provision resources there too

## Create S3 bucket for remote state files

Let us begin preparing out organization master account for use with terraform. Login to your new AWS using your root credentials. (a note about your root credentials, where possible avoid using this user. IAM will allow you delegate permission to users without exposing the riskier behavior made available only to the root account). We will need to provide a remote state repository. Terraform tracks changes to resources using something called a state file. If we don't use a remote repository for persisting this state file, we will need to rely on committing it alongside or terraform code. This is ill-advised, you will see that your [state file is not shy about storing information you might consider sensitive][terraform-data].

1. Navigate to the S3 console by typing in 's3' in the search bar on the homepage of the AWS console and selecting it from the drop down list.
2. Select the orange "Create bucket" button
3. Give a unique bucket name (these must be unique across the global S3 namespace, it to be uniquely named among all buckets that exist in AWS)
4. Select the AWS region in which you want to create this bucket. This decision may have implications depending on your organization (you may need to keep sensitive data in EU for example). The consensus among AWS users (or perhaps the vocal minority at /r/aws) is to avoid us-east-1 if possible because it tends to have issues when compared with regions. That being said, we run our production workloads in us-east-1 without issue.
5. Block all public access for this bucket, we will interface with this bucket only via priveleged API calls to the S3 API.
6. I would enable bucket versioning for this one. This will help you avoid crisis in the event of unexpected changes to a state file which would otherwise prove catastrophic if not very-difficult to recover from. Think about it like this, if terraform manages resources by storing information about them in a remote state, what happens if that state file is lost? You'd be pretty screwed. I'd argue that enabling backup on this bucket is worth it from a peace-of-mind perspective
7. Create a single tag: terraform: false
8. Enable server-side encryption, for our purposes we'll select Amazon S3 key but in the future you may want to leverage your own KMS key if you choose so
![S3 Bucket Encryption](/assets/remote_state_bucket_encryption.PNG)
9. Ignore the object lock settings under the 'Advanced settings' tab
10. Create bucket

[aws-new-account]: https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/
[terraform-install]: https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started
[terraform-data]: https://www.terraform.io/docs/language/state/sensitive-data.html
