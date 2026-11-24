# Cross-Platform RDP with AWS

A beginner-friendly, hands-on guide to connecting to **AWS Windows EC2 instances** using **Remote Desktop Protocol (RDP)** from **Windows**, **macOS**, and **Linux**.

The guides cover the whole process from start to finish: launching the EC2 instance, configuring security groups and key pairs, retrieving the Administrator password, connecting with the right RDP client for your operating system, and troubleshooting common problems.

---

## Getting started

### Step 1: Set up the EC2 instance (everyone)

Follow **[docs/ec2-setup.md](docs/ec2-setup.md)**. It starts from the AWS console search bar and walks you through launching a Windows Server instance, choosing an instance type, creating a key pair, configuring the security group, and retrieving the Administrator password.

### Step 2: Connect from your computer

| Your computer | RDP client | Guide |
|---|---|---|
| **Windows 10 / 11** | Remote Desktop Connection (`mstsc`), built in | [docs/windows-setup.md](docs/windows-setup.md) |
| **macOS** | Windows App (formerly Microsoft Remote Desktop) | [docs/macos-setup.md](docs/macos-setup.md) |
| **Linux** | Remmina (GUI) or FreeRDP `xfreerdp` (CLI) | [docs/linux-setup.md](docs/linux-setup.md) |

Pick the guide for the computer you're connecting *from*.

### Which instance type?

**`t3.medium` (2 vCPU, 4 GiB RAM) is recommended.** The Free Tier `t3.micro` (1 GiB) works for quick tests, but the Windows desktop is very slow. The full comparison is in [EC2 Setup → Choose the instance type](docs/ec2-setup.md#6-choose-the-instance-type).

---

## What you'll learn

- How to launch a **Windows Server** EC2 instance in the AWS console
- What a **key pair** is, and why Windows instances need one to reveal the password
- How to write a **security group** rule that allows RDP only from your IP address
- How to **decrypt the Administrator password**, in the console or with the AWS CLI
- How to connect, share files, and use the clipboard with each platform's **RDP client**
- How to **diagnose and fix** the most common connection errors
- How to **stop or terminate** resources so you aren't billed after you finish

---

## How it works

1. **You launch** a Windows Server EC2 instance and attach an RSA **key pair** to it.
2. **On first boot**, Windows generates a random `Administrator` password, encrypts it with the key pair's public key, and stores it with AWS.
3. **You decrypt** the password using your private key (`.pem`). Only someone who holds that file can read the password.
4. **The security group** acts as a firewall. It allows inbound **TCP port 3389** (RDP) only from your public IP address.
5. **Your RDP client** connects to the instance's public DNS name or IP address on port 3389, and you sign in as `Administrator`.

---

## Prerequisites

- Access to an AWS account with permission to launch EC2 instances. The account is provided for you, so you don't need to sign up.
- A Windows, macOS, or Linux computer with internet access
- A network that allows **outbound TCP 3389**. Some campus and corporate networks block it; if so, try a phone hotspot.
- *(Optional)* The [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) for command-line password retrieval

---

## Repository structure

```text
Cross-Platform-RDP-with-AWS/
├── README.md               ← You are here: overview of the project
└── docs/
    ├── ec2-setup.md        ← Step 1: launch and configure the Windows EC2 instance
    ├── windows-setup.md    ← Connecting from Windows (mstsc)
    ├── macos-setup.md      ← Connecting from macOS (Windows App)
    └── linux-setup.md      ← Connecting from Linux (Remmina / xfreerdp)
```

---

## Key concepts at a glance

| Term | What it means |
|---|---|
| **EC2 instance** | A virtual server running in AWS. Here it runs Windows Server. |
| **AMI** | Amazon Machine Image: the template the instance is created from, for example *Microsoft Windows Server 2022 Base*. |
| **Region** | The geographic location of your resources, for example `ap-southeast-1` (Singapore). Instances, key pairs, and security groups exist only in their own Region. |
| **Key pair** | A public/private key pair. For Windows instances, the private key (`.pem`) decrypts the initial Administrator password. It must be **RSA**. |
| **Security group** | A virtual firewall attached to the instance. Its inbound rules decide who can reach which port. |
| **RDP** | Remote Desktop Protocol: Microsoft's protocol for remote graphical access. It uses **TCP port 3389** by default. |
| **Public DNS / IP** | The internet address of your instance. It **changes** each time you stop and start the instance, unless you attach an **Elastic IP**. |

---

## Security best practices

- **Never allow RDP from `0.0.0.0/0` (Anywhere).** Restrict port 3389 to **My IP**, and update the rule when your IP address changes.
- **Never commit `.pem` files to Git.** This repository's [`.gitignore`](.gitignore) already excludes `*.pem`, `*.ppk`, and `*.rdp` files.
- **Change the Administrator password** after your first login, and store it in a password manager.
- **Don't share your private key.** Anyone who has it can decrypt the password of every instance that uses that key pair.
- **For production setups**, consider avoiding a public RDP port entirely. Use [AWS Systems Manager Fleet Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/fleet-manager.html) (RDP in the browser), a VPN, or a bastion host instead.

---

## Cost and clean-up

Windows `t2.micro` and `t3.micro` instances may be covered by the AWS Free Tier, depending on the account's Free Tier plan. Larger instance types, including the recommended `t3.medium`, are billed per second while they are running, and Windows instances include a licence cost.

When you finish a lab:

- **Stop** the instance to pause compute charges. You still pay a small amount for the disk.
- **Terminate** the instance to delete it and stop all of its charges.
- **Release** any Elastic IP you allocated.

For step-by-step instructions, see [EC2 Setup → Managing your instance](docs/ec2-setup.md#14-managing-your-instance).

---

## Further reading

- [AWS: Connect to your Windows instance using RDP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connecting_to_windows_instance.html)
- [AWS: Amazon EC2 security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [AWS: Amazon EC2 key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
- [Microsoft: Windows App documentation](https://learn.microsoft.com/windows-app/)
- [Remmina](https://remmina.org/) · [FreeRDP](https://www.freerdp.com/)
