# Connecting to an AWS Windows EC2 Instance from macOS

This guide shows you how to connect to a Windows Server EC2 instance from a **Mac** using Remote Desktop Protocol (RDP).

**Time needed:** about 10 minutes
**RDP client used:** **Windows App** by Microsoft (formerly *Microsoft Remote Desktop*), free on the Mac App Store

---

## Table of Contents

1. [Before you start](#1-before-you-start)
2. [Install Windows App](#2-install-windows-app)
3. [Connect with Windows App](#3-connect-with-windows-app)
4. [After your first login](#4-after-your-first-login)
5. [Optional: share folders and the clipboard](#5-optional-share-folders-and-the-clipboard)
6. [Keyboard tips for Mac users](#6-keyboard-tips-for-mac-users)
7. [Troubleshooting](#7-troubleshooting)
8. [When you're finished](#8-when-youre-finished)

---

## 1. Before you start

Make sure you have:

- ✅ A running Windows Server EC2 instance. If you don't have one yet, follow the **[EC2 Setup guide](ec2-setup.md)** first.
- ✅ The instance's **Public DNS** (or public IP)
- ✅ The username **`Administrator`** and the **decrypted password**
- ✅ A Mac running a **recent version of macOS**. Check the Windows App page on the App Store for the exact minimum version.
- ✅ A network that allows **outbound TCP port 3389**. Some campus and office networks block it.

---

## 2. Install Windows App

macOS has no built-in RDP client, so install Microsoft's free one:

1. Open the **App Store**.
2. Search for **Windows App** (publisher: *Microsoft Corporation*).
3. Click **Get**, then **Install**.

> **Note:** In 2024, Microsoft renamed *Microsoft Remote Desktop* to **Windows App**. If you already had Microsoft Remote Desktop, it updated to Windows App automatically, and your saved PCs are kept. Older tutorials that mention "Microsoft Remote Desktop" describe the same app.

---

## 3. Connect with Windows App

1. Open **Windows App**.
2. Click the **+** button in the toolbar and choose **Add PC**.
3. Fill in the form:
   - **PC name:** the instance's Public DNS, for example `ec2-13-250-xx-xx.ap-southeast-1.compute.amazonaws.com`
   - **Credentials:** choose **Add Credentials…**, then enter:
     - **Username:** `Administrator`
     - **Password:** the decrypted password
   - **Friendly name** *(optional)*: `AWS Windows Server`
4. Click **Add**. A tile for the PC appears.
5. Double-click the tile to connect.
6. You'll see a warning that the certificate couldn't be verified. This is expected. EC2 Windows instances use a self-signed certificate. Click **Continue**.
7. The Windows Server desktop opens in a new window. You're connected! 🎉

> **Shortcut:** If you downloaded the `.rdp` file from the AWS **Connect → RDP client** page, double-click it. It opens in Windows App with the address and username already filled in.

---

## 4. After your first login

- **Change the Administrator password.** Inside the remote session, open **PowerShell** and run the command below, then type a new password when prompted:

  ```powershell
  net user Administrator *
  ```

  Then update the saved password in Windows App: right-click the PC tile → **Edit**.
- **Disconnect vs. sign out:** closing the window *disconnects* you, and your apps keep running on the server. To end the session completely, use **Start → Administrator → Sign out**.

---

## 5. Optional: share folders and the clipboard

1. In Windows App, right-click your PC tile and choose **Edit**.
2. Go to the **Folders** tab, tick **Redirect folders**, click **+**, and choose a Mac folder (for example `~/Desktop`).
3. Go to the **Devices & Audio** tab and make sure **Clipboard** is ticked.
4. Click **Save** and reconnect.
5. Inside the instance, open **File Explorer → This PC**. Your Mac folder appears as a network drive.

---

## 6. Keyboard tips for Mac users

| What you want in Windows | Press on your Mac |
|---|---|
| Copy / Paste | `Cmd + C` / `Cmd + V`. Windows App maps these to `Ctrl` by default. |
| Start menu | `Cmd` (acts as the Windows key) |
| Ctrl + Alt + Del | Use **Connection → Send Ctrl+Alt+Del** in the menu bar if your version has it, or press `Ctrl + Option + Delete` (on some keyboards `fn + Ctrl + Option + Delete`) |
| Switch between Mac and remote apps | `Cmd + Tab` works when the remote window is not full screen |

To change keyboard mappings, go to **Windows App → Settings**.

---

## 7. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| *"Unable to connect"* or error code `0x204` | The security group doesn't allow your current IP | Update the RDP rule to **My IP**. See [Update your IP in the security group](ec2-setup.md#update-your-ip-in-the-security-group). |
| Timeout even though the security group is correct | The instance is stopped, has no public IP, or your network blocks port 3389 | Check that the instance is **Running** and has a **Public IPv4 address**. Try a phone hotspot to rule out a network block. |
| *"Your credentials did not work"* | Wrong password or user name | Edit the PC, re-enter `Administrator`, and paste the password again. Watch for trailing spaces. |
| It worked yesterday but not today | The instance was stopped and started, so its public IP changed | Edit the PC and replace the PC name with the new Public DNS. To keep a fixed address, attach an [Elastic IP](ec2-setup.md#13-optional-attach-an-elastic-ip). |
| Display looks blurry on a Retina screen | Scaling settings | Edit the PC → **Display** → enable **Optimize for Retina displays**, or pick a different resolution. |
| Desktop is very slow | The instance type is too small, or bandwidth is low | Lower the resolution in the PC's **Display** settings. Use `t3.medium` or larger (see [instance types](ec2-setup.md#6-choose-the-instance-type)). |

**Test whether port 3389 is reachable.** Run this in Terminal:

```bash
nc -vz <Public-DNS-or-IP> 3389
```

- `succeeded!` means the network path is open. The problem is your credentials or the client.
- A timeout or *"Connection refused"* means something is blocking the connection. Check the security group, the instance state, and your network.

For password and key problems, see [EC2 Setup → Troubleshooting](ec2-setup.md#15-troubleshooting-setup-problems).

---

## 8. When you're finished

**Stop** the instance when you aren't using it, and **terminate** it when you're completely done. See [Managing your instance](ec2-setup.md#stop-start-and-terminate) for details.

---

**Other guides:** [EC2 Setup](ec2-setup.md) · [Windows](windows-setup.md) · [Linux](linux-setup.md) · [Main README](../README.md)
