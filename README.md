<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Access S3 from a VPC

**Project Link:** [View Project](https://nextwork.ai/projects/36d7dc8f-3a9f-51ff-91ae-662077ea9920)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_3e1e79a2)

## Introducing Today's Project!

### What is Amazon VPC?

Amazon VPC (Virtual Private Cloud) is a securely isolated virtual network dedicated to your AWS account that logically mirrors a traditional physical network, and it is useful because it gives you complete control over your cloud networking environment—including IP address ranges, subnet creation, route tables, and network gateways.

By using an Amazon VPC, you can run your AWS resources—such as Amazon EC2 instances—in a secure and isolated network topology that you define. It acts as the foundational perimeter for your cloud architecture, allowing you to separate public-facing resources from sensitive, private back-end infrastructure.

Beyond security isolation, an Amazon VPC is essential because it allows you to configure granular traffic controls using Security Groups and Network ACLs (Access Control Lists) to dictate exactly what inbound and outbound traffic is permitted. Furthermore, you can establish custom internet gateways, NAT devices, and VPN connections to safely bridge your private cloud infrastructure with the outside world or your on-premises data centers, providing the exact flexibility, scalability, and enterprise-grade networking control required for modern cloud deployments.

### How I used Amazon VPC in this project

In today's project, I used Amazon VPC to deploy a secure and isolated cloud network architecture featuring a public subnet and an Amazon EC2 instance, enabling me to test and verify cross-service communication and file management with external services like Amazon S3 using the AWS CLI and IAM access keys.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project was how straightforward it is to securely bridge resources across a VPC boundary and manage external AWS services like Amazon S3 directly from an EC2 instance terminal using simple AWS CLI commands and access keys.

### This project took me...

This project took me about 40 minutes to complete! 

## In the first part of my project...

### Step 1 - Architecture set up

In this step, I will create a custom VPC and launch an Amazon EC2 instance into its public subnet because these resources form the foundational architecture required to test cross-service communication with Amazon S3.

### Step 2 - Connect to my EC2 instance

In this step, I will connect directly to my EC2 instance using EC2 Instance Connect because it provides secure, browser-based terminal access without needing a custom SSH key pair.

### Step 3 - Set up access keys

In this step, I will create and document an IAM access key because my EC2 instance requires explicit security credentials to authenticate and communicate with AWS services like Amazon S3 from outside the management console.

## Architecture set up

I started my project by launching a custom VPC (NextWork-vpc) and an Amazon Linux 2023 EC2 instance (Instance - NextWork VPC Project) in its public subnet with a public IPv4 address enabled and an SSH security group.

I also set up an IAM access key via the IAM console and securely downloaded the credentials .csv file so my EC2 instance can authenticate and connect to external AWS services

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_4334d777)

## Running CLI commands

AWS CLI (Amazon Web Services Command Line Interface) is a powerful, unified open-source tool that allows cloud engineers and developers to manage, configure, and control multiple AWS services directly from the terminal or command line. Instead of relying solely on the visual AWS Management Console, professionals use the AWS CLI to run shell commands, interact with resources like Amazon S3 or EC2, and automate complex cloud workflows using scripts.

I have access to the AWS CLI because it comes pre-installed by default on Amazon Linux 2023 Amazon EC2 instances. When launching virtual machines in the cloud for administrative or testing tasks, having the CLI built right into the operating system environment saves significant setup time. It allows me to immediately begin testing connectivity, running administrative commands, and managing AWS infrastructure right from the command line once proper credentials or authentication methods are established.

The first command I ran was aws s3 ls. This command is used to list the S3 buckets available in your AWS account

The second command I ran was aws configure. This command is used to set up and configure your AWS credentials (such as your Access Key ID and Secret Access Key) and default region on your EC2 instance so it can securely interact with your AWS account

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_e7fa8776)

## Access keys

### Credentials

To set up my EC2 instance to interact with my AWS environment, I configured my credentials using aws configure, entering my Access Key ID, Secret Access Key, and default region (us-east-1)

Access keys are unique identifiers and secret keys used as credentials to authenticate and securely access your AWS account and services via the CLI, SDKs, or APIs

Secret access keys are the secret password component of your AWS credentials, which pair with your Access Key ID to authenticate and securely authorize your applications, scripts, and instances to communicate with AWS services.

When configuring tools like the AWS CLI on an Amazon EC2 instance, you must provide both the Access Key ID (acting as your username) and the secret access key (acting as your password) so that AWS can verify your identity. Because these keys grant direct access to your cloud account and resources, maintaining their security is critical. Unlike passwords you might use on regular websites, secret access keys cannot be recovered if they are lost; if a key is misplaced, you must deactivate it and generate a brand-new pair through the AWS IAM console.

For safety, secret access keys should always be handled with extreme care. They must never be hardcoded directly into application source code, committed to public code repositories like GitHub, or shared in plain text files where unauthorized users could access them. Exposing a secret access key can allow malicious actors to compromise your cloud infrastructure, run unauthorized services, or incur unexpected financial charges on your AWS account. As a best practice, you should always delete or deactivate access keys immediately when they are no longer needed, and rely on more secure alternatives like IAM roles whenever possible.

### Best practice

Although I'm using access keys in this project, a best practice alternative is to use IAM roles attached directly to the EC2 instance so that it can inherit temporary permissions without requiring long-term secret keys to be manually stored.

## In the second part of my project...

### Step 4 - Set up an S3 bucket

In this step, I will create an Amazon S3 bucket and upload two files into it because I need a destination storage target to test cross-service data interaction and file retrieval from my EC2 instance.

### Step 5 - Connecting to my S3 bucket

In this step, I will connect to my S3 bucket from my EC2 instance using the AWS CLI and my configured credentials because I need to verify cross-service communication and test listing and uploading files across VPC boundaries

## Connecting to my S3 bucket

The first command I ran was aws s3 ls. This command is used to list the S3 buckets available in your AWS account

When I ran the command aws s3 ls again, the terminal responded with a list of the S3 buckets available in my AWS account, including nextwork-vpc-project-jasahorro. This indicated that my EC2 instance successfully authenticated using the new access keys and could now communicate with external AWS services.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_4334d778)

## Connecting to my S3 bucket

Another CLI command I ran was aws s3 ls s3://nextwork-vpc-project-jasahorro, which returned a list of the objects stored inside my S3 bucket (including the two .png images I uploaded earlier).

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_4334d779)

## Uploading objects to S3

To upload a new file to my bucket, I first ran the command sudo touch /tmp/test.txt. This command creates an empty temporary file named test.txt inside the /tmp/ directory of your EC2 instance

The second command I ran was aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-jasahorro. This command will copy the temporary text file you just created from your EC2 instance directly into your S3 bucket.

The third command I ran was aws s3 ls s3://nextwork-vpc-project-jasahorro, which validated that the newly created test.txt file was successfully uploaded to my S3 bucket from the EC2 instance

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/36d7dc8f-3a9f-51ff-91ae-662077ea9920_3e1e79a2)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/36d7dc8f-3a9f-51ff-91ae-662077ea9920)*
