# EC2 Setup: Launching a Windows Server Instance for RDP

This guide walks you through creating a **Windows Server EC2 instance** that you can connect to with Remote Desktop. Complete it **once**. Then follow the guide for the computer you're connecting from: [Windows](windows-setup.md), [macOS](macos-setup.md), or [Linux](linux-setup.md).

**Time needed:** about 15 minutes, plus a few minutes for Windows to boot

**What you'll have at the end:**

- A running Windows Server instance
- Its **Public DNS** name (the address you connect to)
- The **Administrator password**
- A private key file (`.pem`) saved safely on your computer

---

## Table of Contents

1. [Before you begin](#1-before-you-begin)
2. [Open EC2 from the search bar](#2-open-ec2-from-the-search-bar)
3. [Check the Region](#3-check-the-region)
4. [Start launching an instance](#4-start-launching-an-instance)
5. [Choose the Windows AMI](#5-choose-the-windows-ami)
6. [Choose the instance type](#6-choose-the-instance-type)
7. [Create a key pair](#7-create-a-key-pair)
8. [Configure network settings and the security group](#8-configure-network-settings-and-the-security-group)
9. [Configure storage](#9-configure-storage)
10. [Launch the instance](#10-launch-the-instance)
11. [Wait for the instance to be ready](#11-wait-for-the-instance-to-be-ready)
12. [Get the connection details and password](#12-get-the-connection-details-and-password)
13. [Optional: attach an Elastic IP](#13-optional-attach-an-elastic-ip)
14. [Managing your instance](#14-managing-your-instance)
15. [Troubleshooting setup problems](#15-troubleshooting-setup-problems)

---

## 1. Before you begin

- **AWS access is provided for you.** You don't need to create an AWS account. Sign in with the details you were given, and once you see the AWS Management Console home page, start at [step 2](#2-open-ec2-from-the-search-bar).
- **You're probably sharing an AWS account.** Other people may launch instances in the same account, so give every resource a name that includes **your own name**, for example `alex-windows-rdp`. That way you can always tell which instance, key pair, and security group belong to you.
- **Only touch your own resources.** Never stop, terminate, or edit an instance, key pair, or security group that you didn't create.

---

## 2. Open EC2 from the search bar

1. Click the **search bar** at the top of the AWS console. You can also press `Alt + S` on Windows and Linux, or `Option + S` on macOS.
2. Type **`EC2`**.
3. Under **Services**, click **EC2** (*Virtual Servers in the Cloud*).

The **EC2 Dashboard** opens.

> **Tip:** Click the ☆ star next to **EC2** in the search results to pin it to the console's navigation bar for next time.

---

## 3. Check the Region

Look at the **Region** name in the top-right corner of the console, for example *Asia Pacific (Singapore)*.

- If you were told which Region to use, make sure that Region is selected.
- Otherwise, choose one close to you, such as **Asia Pacific (Singapore) `ap-southeast-1`**. A nearby Region means lower latency and a smoother remote desktop.

> **Important:** Instances, key pairs, and security groups exist **only in the Region where you create them**. If your instance seems to have disappeared later, you are almost certainly looking at a different Region.

---

## 4. Start launching an instance

1. On the EC2 Dashboard, click the orange **Launch instance** button.
2. Under **Name and tags**, enter a name such as **`<yourname>-windows-rdp`**.

---

## 5. Choose the Windows AMI

An **AMI** (Amazon Machine Image) is the operating system template for your instance.

1. Under **Application and OS Images (Amazon Machine Image)**, click the **Windows** tile.
2. In the AMI dropdown, choose **Microsoft Windows Server 2022 Base**. **Microsoft Windows Server 2025 Base** also works.

| ✅ Choose | ❌ Avoid | Why |
|---|---|---|
| `Microsoft Windows Server 2022 Base` | `... Core Base` | **Core** has no graphical desktop. RDP opens a command prompt only. |
| `Microsoft Windows Server 2025 Base` | `... with SQL Server ...` | These include a paid SQL Server licence, which costs much more. |

> **Note:** The AMI name is shown directly under the dropdown. Double-check that it says **Base** and does **not** say *Core*.

---

## 6. Choose the instance type

The instance type decides how much CPU and memory the server gets. Windows Server with a desktop needs much more memory than Linux.

| Instance type | vCPU | Memory | Recommendation |
|---|---|---|---|
| `t3.micro` | 2 | 1 GiB | Free Tier eligible, but **very slow**. Windows boots and you can connect, but opening a browser or Server Manager can take a minute. Use only for quick tests. |
| `t3.small` | 2 | 2 GiB | Budget option. This is the minimum memory Microsoft recommends for Windows Server with a desktop. Fine for light tasks. |
| **`t3.medium`** | **2** | **4 GiB** | ⭐ **Recommended.** A smooth desktop for browsing, installing software, and general lab work. |
| `t3.large` | 2 | 8 GiB | For heavier work: an IDE, SQL Server Express, many browser tabs. |

**Recommendation: use `t3.medium`.** It's the best balance of cost and usability for a Windows desktop over RDP.

To select it:

1. Under **Instance type**, click the dropdown.
2. Type `t3.medium` and select it.

> **Cost note:** Every type except `t3.micro` is billed per second while the instance is **running**, and Windows instances include the Windows licence in the hourly price. Check current prices on the [EC2 On-Demand pricing page](https://aws.amazon.com/ec2/pricing/on-demand/) (choose your Region and *Windows*). **Stop the instance when you aren't using it.**

> **Why `t3` and not `t2`?** `t3` is the newer generation. It's typically a little cheaper and faster than `t2` for the same size. If `t3` isn't available in your Region or account, the matching `t2` size works the same way.

> **About T3 "Unlimited" mode:** T3 instances can burst above their baseline CPU. If the CPU stays very high for a long time, AWS may add a small extra charge. This doesn't happen in normal desktop use.

---

## 7. Create a key pair

Windows instances don't use the key pair to log in directly. Instead, AWS **encrypts the Administrator password** with the key pair, and you need the private key file to **decrypt** it.

1. Under **Key pair (login)**, click **Create new key pair**.
2. Fill in the form:
   - **Key pair name:** `<yourname>-rdp-key`
   - **Key pair type:** **RSA**. ⚠️ *ED25519 does not work for Windows password decryption.*
   - **Private key file format:** **.pem**
3. Click **Create key pair**. The browser downloads `<yourname>-rdp-key.pem`.
4. Check that the new key pair is selected in the **Key pair** dropdown.

### Save the key file safely

AWS lets you download the private key **only once**. Move it out of your Downloads folder now.

**Windows (PowerShell):**

```powershell
mkdir $HOME\.ssh -Force
Move-Item $HOME\Downloads\<yourname>-rdp-key.pem $HOME\.ssh\
```

**macOS / Linux (Terminal):**

```bash
mkdir -p ~/.ssh
mv ~/Downloads/<yourname>-rdp-key.pem ~/.ssh/
chmod 400 ~/.ssh/<yourname>-rdp-key.pem
```

> **Warning:** If you lose this file, you can't recover the password of any instance that uses this key. Never share it, email it, or commit it to Git.

---

## 8. Configure network settings and the security group

A **security group** is a virtual firewall around your instance. You need one rule that allows **RDP (TCP port 3389)** from **your IP address only**.

1. In the **Network settings** panel, click **Edit**, at the top-right of the panel.
2. Check these settings:
   - **VPC:** leave the default.
   - **Subnet:** *No preference* is fine.
   - **Auto-assign public IP:** **Enable**. ⚠️ *Without a public IP, you can't reach the instance from the internet.*
3. **Firewall (security groups):** select **Create security group**.
   - **Security group name:** `<yourname>-rdp-sg`
   - **Description:** `RDP access for <yourname>`
4. Under **Inbound security group rules**, the console usually adds an RDP rule for you. Make sure it looks like this:

| Type | Protocol | Port range | Source type | Source |
|---|---|---|---|---|
| RDP | TCP | 3389 | **My IP** | *(filled in automatically)* |

If there's no rule, click **Add security group rule** and fill it in as shown above.

To confirm the IP address that the console filled in, check your public IP:

```bash
curl https://checkip.amazonaws.com
```

On Windows, use `curl.exe https://checkip.amazonaws.com` in PowerShell.

> **Never set the source to *Anywhere* (`0.0.0.0/0`).** An RDP port open to the whole internet is found by automated scanners within minutes, and they will start trying to guess the Administrator password.

> **Your IP can change.** Home Wi-Fi, campus networks, and mobile hotspots often change your public IP. If you can't connect later, update this rule. See [Update your IP in the security group](#update-your-ip-in-the-security-group).

---

## 9. Configure storage

Leave the default: **30 GiB, gp3**. That's enough for Windows Server plus a few applications.

---

## 10. Launch the instance

1. Review the **Summary** panel on the right:
   - **Software image:** Microsoft Windows Server 2022 Base (or 2025 Base)
   - **Instance type:** `t3.medium` (or the type you chose)
   - **Firewall:** your new security group
   - **Storage:** 30 GiB
2. Click **Launch instance**.
3. When you see the green **Success** banner, click **View all instances**.

---

## 11. Wait for the instance to be ready

In the **Instances** list, find your instance by its name and watch two columns:

| Column | Wait until it shows |
|---|---|
| **Instance state** | 🟢 **Running** |
| **Status check** | ✅ **3/3 checks passed** (some consoles show *2/2*) |

Click the 🔄 refresh button to update the list.

> **Be patient:** Windows takes **4–10 minutes** to finish its first boot. AWS can show the password only after that. If you try too early, you'll see *"Password is not available yet."*

---

## 12. Get the connection details and password

1. Tick the checkbox next to your instance.
2. Click **Connect** at the top of the page.
3. Open the **RDP client** tab.
4. Write down:
   - **Public DNS**, for example `ec2-13-250-xx-xx.ap-southeast-1.compute.amazonaws.com`
   - **Username:** `Administrator`
5. Click **Get password**.
6. Click **Upload private key file** and select your `.pem` file from the `.ssh` folder where you saved it.
   - **Windows:** in the file dialog, type `%USERPROFILE%\.ssh` in the address bar.
   - **macOS:** press `Cmd + Shift + G` and type `~/.ssh`. You can also press `Cmd + Shift + .` to show hidden folders.
   - **Linux:** press `Ctrl + L` and type `~/.ssh`. You can also press `Ctrl + H` to show hidden folders.
7. Click **Decrypt password**.
8. Copy the **password** and store it somewhere safe, such as a password manager.

> **Tip:** The **Download remote desktop file** button on the same page saves a `.rdp` file with the address and username already filled in. Windows and macOS can open it directly.

<details>
<summary><strong>Optional: get the password with the AWS CLI</strong></summary>

If you have the [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with your access keys, run:

```bash
aws ec2 get-password-data \
  --instance-id i-0123456789abcdef0 \
  --priv-launch-key ~/.ssh/<yourname>-rdp-key.pem \
  --region ap-southeast-1 \
  --query PasswordData --output text
```

Replace the instance ID with yours. You'll find it in the **Instance ID** column. In PowerShell, use a backtick (`` ` ``) instead of `\` at the end of each line.

</details>

**You're ready to connect.** Continue with the guide for your computer:

| Your computer | Guide |
|---|---|
| Windows | [windows-setup.md](windows-setup.md) |
| macOS | [macos-setup.md](macos-setup.md) |
| Linux | [linux-setup.md](linux-setup.md) |

---

## 13. Optional: attach an Elastic IP

By default, your instance gets a **new public IP and DNS name every time you stop and start it**. To keep a fixed address:

1. In the left menu, go to **Network & Security → Elastic IPs**.
2. Click **Allocate Elastic IP address**, then **Allocate**.
3. Select the new address, then choose **Actions → Associate Elastic IP address**.
4. Choose your instance and click **Associate**.

> **Cost note:** AWS charges for public IPv4 addresses, including Elastic IPs, so **release** the Elastic IP when you finish. Use **Actions → Disassociate**, then **Actions → Release**.

---

## 14. Managing your instance

### Update your IP in the security group

If your connection times out and your public IP has changed:

1. Go to **EC2 → Network & Security → Security Groups**.
2. Select `<yourname>-rdp-sg`.
3. Open the **Inbound rules** tab and click **Edit inbound rules**.
4. In the RDP rule's **Source** dropdown, choose **My IP** again.
5. Click **Save rules**.

### Stop, start, and terminate

Select your instance, then open the **Instance state** menu:

| Action | What happens | Billing |
|---|---|---|
| **Stop instance** | Shuts Windows down. Your files are kept. | Compute charges stop. You still pay a small amount for the 30 GiB disk. |
| **Start instance** | Boots it again. The **public IP and DNS change**, unless you use an Elastic IP. | Compute charges resume. |
| **Reboot instance** | Restarts Windows. The IP stays the same. | No change. |
| **Terminate instance** | **Permanently deletes** the instance and its disk. This can't be undone. | All charges for it stop. |

> **When you're completely finished:** terminate the instance, delete your security group (`<yourname>-rdp-sg`), and release any Elastic IP you allocated. You can keep the key pair and reuse it for future instances.

---

## 15. Troubleshooting setup problems

| Problem | Likely cause | Fix |
|---|---|---|
| *"You are not authorized to perform this operation"* | Your user doesn't have permission for that action or instance type | Ask the person who provided the AWS account to grant access, or try an instance type they allow. |
| Can't find your instance | You're in a different Region | Switch the Region in the top-right corner to the one you used. |
| *"Password is not available yet"* | Windows hasn't finished its first boot | Wait 4–10 minutes after launch, then try again. |
| *"Private key is invalid"*, or decryption fails | You uploaded the wrong `.pem`, or the key type is ED25519 | Check the **Key pair assigned at launch** in the instance's **Details** tab and use that file. If the key was ED25519, terminate the instance and launch a new one with an **RSA** key. |
| Lost the `.pem` file | It can only be downloaded once | For a fresh lab instance, the simplest fix is to terminate it and launch a new one with a new key pair. |
| No **Public IPv4 address** or **Public DNS** shown | **Auto-assign public IP** was disabled | Terminate the instance and launch again with **Auto-assign public IP: Enable**. You can also attach an [Elastic IP](#13-optional-attach-an-elastic-ip). |
| RDP opens only a black command prompt | You chose a **Core** AMI | Terminate it and launch again with a **Base** AMI. |
| *"InsufficientInstanceCapacity"* when launching | AWS has no spare capacity for that type in that zone | Try again in a few minutes, or set a specific subnet or Availability Zone in **Network settings**. |

For connection problems, such as timeouts, login failures, and certificate warnings, see the **Troubleshooting** section of your OS guide.

---

[← Back to the main README](../README.md)
