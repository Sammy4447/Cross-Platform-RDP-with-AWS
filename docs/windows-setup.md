# Connecting to an AWS Windows EC2 Instance from Windows

This guide shows you how to connect to a Windows Server EC2 instance from a **Windows** computer using Remote Desktop Protocol (RDP).

**Time needed:** about 10 minutes
**RDP client used:** Remote Desktop Connection (`mstsc.exe`), which comes with Windows

---

## Table of Contents

1. [Before you start](#1-before-you-start)
2. [Check Remote Desktop Connection](#2-check-remote-desktop-connection)
3. [Connect with Remote Desktop Connection](#3-connect-with-remote-desktop-connection)
4. [After your first login](#4-after-your-first-login)
5. [Optional: share files and the clipboard](#5-optional-share-files-and-the-clipboard)
6. [Alternative client: Windows App](#6-alternative-client-windows-app)
7. [Troubleshooting](#7-troubleshooting)
8. [When you're finished](#8-when-youre-finished)

---

## 1. Before you start

Make sure you have:

- ✅ A running Windows Server EC2 instance. If you don't have one yet, follow the **[EC2 Setup guide](ec2-setup.md)** first.
- ✅ The instance's **Public DNS** (or public IP)
- ✅ The username **`Administrator`** and the **decrypted password**
- ✅ A computer running **Windows 10 or Windows 11**. Remote Desktop Connection is included in every edition, including Home.
- ✅ A network that allows **outbound TCP port 3389**. Some campus and office networks block it.

---

## 2. Check Remote Desktop Connection

Remote Desktop Connection is already installed. To check:

1. Press `Win + R`.
2. Type `mstsc` and press **Enter**.

The **Remote Desktop Connection** window should open. You can also find it by searching **Remote Desktop Connection** in the Start menu.

---

## 3. Connect with Remote Desktop Connection

1. In Remote Desktop Connection, click **Show Options**.
2. On the **General** tab, fill in:
   - **Computer:** the instance's Public DNS, for example `ec2-13-250-xx-xx.ap-southeast-1.compute.amazonaws.com`
   - **User name:** `Administrator`
3. *(Optional)* On the **Display** tab, pick a smaller resolution if your connection is slow.
4. *(Optional)* Back on the **General** tab, click **Save As…** to save these settings as a `.rdp` file for next time.
5. Click **Connect**.
6. When asked for credentials, paste the **decrypted password** and click **OK**.
7. You'll see a warning: *"The identity of the remote computer cannot be verified."* This is expected. EC2 Windows instances use a self-signed certificate. Click **Yes** to continue.
8. The Windows Server desktop appears. You're connected! 🎉

> **Shortcut:** If you downloaded the `.rdp` file from the AWS **Connect → RDP client** page, just double-click it. The address and username are already filled in.

---

## 4. After your first login

- **Change the Administrator password:** press `Ctrl + Alt + End` (the RDP version of `Ctrl + Alt + Del`) and choose **Change a password**.
- **Disconnect vs. sign out:** closing the RDP window *disconnects* you, and your apps keep running on the server. To end the session completely, use **Start → Administrator → Sign out**.
- **Leave full screen:** use the blue connection bar at the top of the screen, or press `Ctrl + Alt + Break`.

---

## 5. Optional: share files and the clipboard

To copy files between your PC and the EC2 instance:

1. In Remote Desktop Connection, click **Show Options → Local Resources**.
2. Make sure **Clipboard** is ticked. You can then copy and paste text, and even files, with `Ctrl + C` and `Ctrl + V`.
3. Under **Local devices and resources**, click **More…**, expand **Drives**, and tick the drive you want to share (for example `C:`).
4. Click **OK**, then **Connect**.
5. Inside the instance, open **File Explorer → This PC**. Your shared drive appears as *"C on YOUR-PC"*.

---

## 6. Alternative client: Windows App

Microsoft's newer **Windows App** (from the Microsoft Store) also supports RDP:

1. Install **Windows App** from the Microsoft Store.
2. Click **+ → Add PC**.
3. Enter the Public DNS as the **PC name**.
4. Add a user account with the username `Administrator` and your password.
5. Save the PC, then double-click it to connect.

The built-in `mstsc` works the same and needs no installation, so this guide uses it.

---

## 7. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| *"Remote Desktop can't connect to the remote computer"* or a timeout | The security group doesn't allow your current IP | Update the RDP rule to **My IP**. See [Update your IP in the security group](ec2-setup.md#update-your-ip-in-the-security-group). |
| Timeout even though the security group is correct | The instance is stopped, has no public IP, or your network blocks port 3389 | Check that the instance is **Running** and has a **Public IPv4 address**. Try a phone hotspot to rule out a network block. |
| *"The logon attempt failed"* | The password was mistyped or has extra spaces | Copy and paste the password again. Use `Administrator` as the user name, not your own name. |
| It worked yesterday but not today | The instance was stopped and started, so its public IP changed | Copy the new Public DNS from the EC2 console. To keep a fixed address, attach an [Elastic IP](ec2-setup.md#13-optional-attach-an-elastic-ip). |
| *"An authentication error has occurred… CredSSP encryption oracle remediation"* | Your PC and the server are at different update levels | Run Windows Update on your PC. This is rare with current AMIs. |
| Desktop is very slow | The instance type is too small, or bandwidth is low | Lower the resolution on the **Display** tab and the colour depth on the **Experience** tab. Use `t3.medium` or larger (see [instance types](ec2-setup.md#6-choose-the-instance-type)). |

**Test whether port 3389 is reachable.** Run this in PowerShell:

```powershell
Test-NetConnection -ComputerName <Public-DNS-or-IP> -Port 3389
```

- `TcpTestSucceeded : True` means the network path is open. The problem is your credentials or the client.
- `TcpTestSucceeded : False` means something is blocking the connection. Check the security group, the instance state, and your network.

For password and key problems, see [EC2 Setup → Troubleshooting](ec2-setup.md#15-troubleshooting-setup-problems).

---

## 8. When you're finished

**Stop** the instance when you aren't using it, and **terminate** it when you're completely done. See [Managing your instance](ec2-setup.md#stop-start-and-terminate) for details.

---

**Other guides:** [EC2 Setup](ec2-setup.md) · [macOS](macos-setup.md) · [Linux](linux-setup.md) · [Main README](../README.md)
