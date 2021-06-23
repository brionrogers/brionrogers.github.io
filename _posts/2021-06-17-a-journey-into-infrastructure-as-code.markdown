---
layout: post
title:  "A Journey Into Infrastructure As Code"
date:   2021-06-17 23:17:45 -0400
categories: aws terraform iac packer hashicorp
---
If you build any system of moderate complexity in the cloud, whether public or private, you will quickly observe that the amount of effort required to effectively administer the system scales according to its' complexity. Simply deprovisioning unused resources can become a non-trivial task at scale, never mind more complex tasks which would require a review prior to implementation. The fact is, administering cloud environments manually isn't just impractical at scale, it's objectively dangerous in a production environment. Configuration drift, one-off changes, context-less records in CloudTrail -- avoiding these kinds of issues while performing manual administration of an environment requires real discipline on the part of the operations team.

What if we could avoid all that mess? This is where infrastructure as code (IaC) comes in. Infrastructure as code, as the name implies, involves describing your environment using some kind of coding language in such a way that the infrastructure you are representing can be modified indirectly through changes to the underlying code. There are a number of solutions to implement IaC ranging from platform independent options such as Terraform that can provision both cloud and on-prem based resources to cloud-provider specific flavors like Cloudformation or AzureRM templates.

My personal preference is Terraform, it was a more mature offering before many of the current solutions put forth by the major cloud providers. It has easy syntax, excellent CLI tools, and there are some nice tools out there for it i.e. terragrunt (truly there are some wild ones out there)

While it is easy enough to get started with Terraform, you can find yourself in a trap if you don't make the correct decisions early on in your project. The types of issues that will bite you are mostly related to the layout of your project structure when dealing with multiple logically isolated accounts. These issues are largely mitigated through thoughtful choices early on about how you intend to store and secure your state file as well as how you intend to manage multiple environments which are theoretically mirror copies of each other. (luckily Terraform has workspaces which we can use to deal with these issues)

I will attempt to layout:
* a viable way to organize a terraform project
* how to use terraform workspaces to create mirror copies of environments
* my thoughts on isolating state changes safely and blast radius
