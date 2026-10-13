# Access S3 from a VPC — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md).

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [Getting into the instance: EC2 Instance Connect](#getting-into-the-instance-ec2-instance-connect)
4. [The AWS CLI](#the-aws-cli)
5. [Credentials and access keys](#credentials-and-access-keys)
6. [Access keys vs IAM roles](#access-keys-vs-iam-roles)
7. [Configuring the CLI](#configuring-the-cli)
8. [S3 commands I ran](#s3-commands-i-ran)
9. [VPC endpoints (the private route)](#vpc-endpoints-the-private-route)
10. [Build order and cleanup](#build-order-and-cleanup)
11. [Self-quiz](#self-quiz)

---

## What I built

- 🌐 **A custom VPC** (`NextWork-vpc`) with a public subnet.
- 💻 **An EC2 instance** (Amazon Linux 2023) in that public subnet, with a public IPv4 address and a security group that allows SSH.
- 🔑 **An IAM access key**, entered into the instance with `aws configure` so the instance could make calls to my AWS account.
- 🪣 **An S3 bucket** (`nextwork-vpc-project-jasahorro`). From the instance's terminal I listed the bucket, listed its objects and uploaded a new file to it.

```
  Me (browser) ──EC2 Instance Connect (SSH)──►  ┌──────────── NextWork-vpc ────────────┐
                                                │  public subnet                       │
                                                │  ┌──────────────────────────┐        │
                                                │  │ EC2 (Amazon Linux 2023)  │        │
                                                │  │ AWS CLI + access key     │        │
                                                │  └────────────┬─────────────┘        │
                                                └───────────────┼──────────────────────┘
                                                                │ internet gateway
                                                                ▼ (public internet)
                                                  ┌──────────────────────────────┐
                                                  │ S3: nextwork-vpc-project-... │
                                                  └──────────────────────────────┘
```

In this project the traffic to S3 goes **out over the public internet**. A **VPC endpoint** would keep it inside the AWS network instead (see [VPC endpoints](#vpc-endpoints-the-private-route)).

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **VPC** | Your own isolated network inside AWS | A private office building you design yourself |
| **Public subnet** | A subnet whose route table sends internet traffic to an internet gateway | The ground-floor rooms with doors to the street |
| **EC2 Instance Connect** | Browser-based terminal access to an EC2 instance | A remote-desktop button for your server |
| **AWS CLI** | A tool for controlling AWS services by typing commands in a terminal | A TV remote for AWS, instead of walking to the TV (the console) |
| **Credentials** | Proof of who you are, so AWS lets you act on your account | An ID badge that gets you through the doors |
| **Access key ID** | The "username" part of an access key | The name printed on the badge |
| **Secret access key** | The "password" part that pairs with the access key ID | The PIN for the badge, which only you should know |
| **IAM role** | A set of permissions that an instance can take on, with temporary credentials | A visitor's lanyard handed out at the desk and taken back when they leave |
| **Key pair** | Public/private keys for logging in to an EC2 instance over SSH | The key to the server's front door (not to your AWS account) |
| **S3 bucket** | A container for storing files (objects) in Amazon S3 | A labelled storage locker |
| **VPC endpoint** | A private connection from your VPC to an AWS service | A private hallway to the locker room, so you don't have to go outside |

## Getting into the instance: EC2 Instance Connect

- EC2 Instance Connect opens the **instance's terminal** in the browser. From there you can install software or call other AWS services.
- It needs the instance to be **in a public subnet** *and* to **have a public IPv4 address**.
- The new security group allows **inbound SSH (port 22)** by default. That's all Instance Connect needs.
- Earlier projects also added an **ICMP** inbound rule, which is only needed for `ping` tests. This project has none, so the rule was skipped.

## The AWS CLI

- The **AWS CLI** controls AWS services from the terminal instead of clicking around the console.
- It comes **pre-installed on Amazon Linux** instances (not on every AMI). You can also install it on your own computer.
- **Console vs CLI:** the console is great for learning because you can see everything. The CLI is faster, and you can **script and automate** it, which is why professionals use it so much.
- The first `aws s3 ls` **failed** because the instance had **no credentials** yet.

## Credentials and access keys

- When you log in to the **console**, you're already authenticated as your IAM user.
- An **EC2 instance is like another user**. To call AWS it needs its own credentials, and it has none by default.
- An **access key** = **access key ID** (like a username) + **secret access key** (like a password). You need both.
- You create access keys in **IAM** (IAM → Users → your user → Security credentials). They belong to that **IAM user**, so the instance gets that user's permissions.
- The secret is **only shown once**, when you create it. If you lose it, you can't get it back, so you delete the key and create a new one.

> **Never** put access keys in code, push them to GitHub or share them in a plain text file. Anyone who has them can use your AWS account and run up charges. Delete them as soon as you're done.

**Access keys vs key pairs** (easy to mix up):

| | Key pair | Access key |
|---|---|---|
| Used for | Logging in **to an EC2 instance** over SSH | Letting apps and servers call **AWS services** (S3, EC2, ...) |
| Made in | EC2 console, when launching an instance | IAM console |
| You keep | The private `.pem` file | The access key ID + secret access key (`.csv`) |
| Analogy | The key to one server's front door | Your badge for the whole AWS building |

**The use-case options in IAM's "Create access key" screen:**

| Option | Pick it when |
|---|---|
| **Command Line Interface (CLI)** | You'll use the key with the AWS CLI (this project) |
| **Local code** | An app you're building on your own computer needs to call AWS |
| **Application running on an AWS compute service** | Your app runs on EC2, Lambda and so on (AWS recommends a role here instead) |
| **Third-party service** | Another company's product needs access to your account (for example, CrowdStrike for security monitoring) |
| **Application running outside AWS** | Your own app runs on a physical server or another cloud |

## Access keys vs IAM roles

- The **yellow warning banner** when creating the key says the same thing as best practice: for EC2, **attach an IAM role** instead of typing in access keys.
- With a role, the instance **takes on the role's permissions** and gets **temporary credentials that rotate automatically**. There's no long-term secret saved on the instance.
- You can attach or detach a role at any time to grant or remove access.
- This project used access keys **only to learn how they work**.

## Configuring the CLI

`aws configure` asks four questions:

| Prompt | What I entered |
|---|---|
| AWS Access Key ID | The key ID from the `.csv` |
| AWS Secret Access Key | The secret from the `.csv` |
| Default region name | `us-east-1` |
| Default output format | `json` (or press Enter, since JSON is the default) |

**Output formats:**

| Format | What it looks like |
|---|---|
| `json` | The default. Structured data that's easy for both people and programs to read |
| `text` | Plain text, handy for piping into other commands |
| `table` | A formatted table, easiest to read at a glance |
| `yaml` | A tidy, human-friendly data format |

> `aws configure` saves the keys in plain text in `~/.aws/credentials` on the instance. That's another reason to **terminate the instance and delete the key** when you're finished.

## S3 commands I ran

| Command | What it does |
|---|---|
| `aws s3 ls` | Lists all the S3 buckets the credentials can see |
| `aws s3 ls s3://nextwork-vpc-project-jasahorro` | Lists the objects inside that bucket |
| `sudo touch /tmp/test.txt` | Creates an empty file called `test.txt` in `/tmp` |
| `aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-jasahorro` | Copies the file from the instance into the bucket (source first, then destination) |
| `aws s3 ls s3://nextwork-vpc-project-jasahorro` | Run again to confirm that `test.txt` arrived |

**Breaking down `sudo touch /tmp/test.txt`:**
- **`sudo`** ("superuser do"): runs the command with admin permissions, which you need for things like installing software. *It isn't actually needed here, because anyone can write to `/tmp`.*
- **`touch`**: creates an empty file if it doesn't exist. If the file does exist, it updates its timestamps instead.
- **`/tmp/`**: a folder for temporary files.
- **`test.txt`**: the file's name.

**The `0` next to `test.txt`** in the listing is the file size in bytes. The file is empty, so it's 0.

## VPC endpoints (the private route)

- **Without an endpoint** (this project): the instance reaches S3 through the **internet gateway over the public internet**.
- **With a VPC endpoint**: the VPC connects to S3 **privately**, and the traffic stays inside the AWS network.
- Benefits: **better security** (no public internet path, so it even works for instances in private subnets) and **lower data transfer costs** (for example, you don't need a NAT gateway).
- This is the follow-up topic after this project.

## Build order and cleanup

1. Create the VPC (`NextWork-vpc`) with a public subnet.
2. Launch an Amazon Linux 2023 EC2 instance in the public subnet, with **auto-assign public IPv4 enabled** and a security group that allows SSH.
3. Connect with **EC2 Instance Connect**, then run `aws s3 ls`. It fails because there are no credentials.
4. In **IAM**, create an access key (use case: CLI) and download the `.csv`.
5. Run `aws configure` and enter the key ID, the secret, `us-east-1` and `json`.
6. Create the S3 bucket and upload two files to it.
7. Run `aws s3 ls`, then `aws s3 ls s3://<bucket>`, then `touch` + `aws s3 cp`, then `ls` again to confirm.

**Cleanup:**
- **Deactivate and delete the access key in IAM.** This is the most important step, because the key keeps working until you delete it.
- Delete the downloaded `accessKeys.csv`.
- Empty the S3 bucket, then delete it.
- Terminate the EC2 instance.
- Delete the VPC, which also removes its subnets, route tables and internet gateway.

---

## Self-quiz

<details><summary>1. What two things does EC2 Instance Connect need from the instance?</summary>

The instance has to be in a **public subnet** and have a **public IPv4 address**. Its security group also has to allow SSH.
</details>

<details><summary>2. Why didn't this project need an ICMP inbound rule?</summary>

ICMP is only needed for `ping` tests, and this project didn't run any.
</details>

<details><summary>3. What is the AWS CLI, and why do engineers use it over the console?</summary>

It's a tool for controlling AWS services from the terminal. It's faster than the console, and you can script and automate it.
</details>

<details><summary>4. Why did the first `aws s3 ls` fail?</summary>

The EC2 instance had no credentials, so AWS didn't know who was asking.
</details>

<details><summary>5. What are the two parts of an access key, and what do they compare to?</summary>

The **access key ID** (like a username) and the **secret access key** (like a password).
</details>

<details><summary>6. What's the difference between a key pair and an access key?</summary>

A key pair logs you in **to an EC2 instance** over SSH. An access key lets apps and servers call **AWS services** through the CLI, SDKs or APIs.
</details>

<details><summary>7. You lost your secret access key. What do you do?</summary>

You can't recover it, because it's only shown once. Deactivate and delete that key, then create a new one in IAM.
</details>

<details><summary>8. What's the best-practice alternative to putting access keys on an EC2 instance, and why?</summary>

Attach an **IAM role**. The instance gets temporary credentials that rotate automatically, there's no long-term secret saved on it, and you can attach or detach the role at any time.
</details>

<details><summary>9. Which "Create access key" option fits an app built by another company that needs access to your AWS account?</summary>

**Third-party service** (for example, CrowdStrike).
</details>

<details><summary>10. What four things does `aws configure` ask for?</summary>

The access key ID, the secret access key, the default region (`us-east-1`) and the default output format (`json` by default).
</details>

<details><summary>11. Name the four CLI output formats.</summary>

`json` (the default), `text`, `table` and `yaml`.
</details>

<details><summary>12. In `aws s3 cp /tmp/test.txt s3://my-bucket`, which part is the source and which is the destination?</summary>

`/tmp/test.txt` is the source (on the instance), and `s3://my-bucket` is the destination (the bucket).
</details>

<details><summary>13. What does `touch` do if the file already exists?</summary>

It doesn't change the contents. It just updates the file's timestamps.
</details>

<details><summary>14. What does the `0` next to `test.txt` in the bucket listing mean?</summary>

The file size is 0 bytes, because the file is empty.
</details>

<details><summary>15. How did traffic from the instance reach S3 in this project, and what would keep it private?</summary>

It went out through the internet gateway over the public internet. A **VPC endpoint** would keep it inside the AWS network, which is more secure and can cost less.
</details>

<details><summary>16. What's the single most important cleanup step, and why?</summary>

**Delete the access key in IAM.** Deleting the `.csv` or the instance doesn't stop the key from working. Only deleting it in IAM does.
</details>
