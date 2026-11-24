# Connecting to an AWS Windows EC2 Instance from Linux

This guide shows you how to connect to a Windows Server EC2 instance from a **Linux** computer using Remote Desktop Protocol (RDP).

**Time needed:** about 10 minutes
**RDP clients used:**
- **Remmina**: a graphical client, recommended for beginners
- **FreeRDP (`xfreerdp`)**: a command-line client, for those comfortable with the terminal

---

## Table of Contents

1. [Before you start](#1-before-you-start)
2. [Install an RDP client](#2-install-an-rdp-client)
3. [Option A: connect with Remmina (GUI)](#3-option-a-connect-with-remmina-gui)
4. [Option B: connect with xfreerdp (CLI)](#4-option-b-connect-with-xfreerdp-cli)
5. [After your first login](#5-after-your-first-login)
6. [Optional: share folders and the clipboard](#6-optional-share-folders-and-the-clipboard)
7. [Troubleshooting](#7-troubleshooting)
8. [When you're finished](#8-when-youre-finished)

---

## 1. Before you start

Make sure you have:

- ✅ A running Windows Server EC2 instance. If you don't have one yet, follow the **[EC2 Setup guide](ec2-setup.md)** first.
- ✅ The instance's **Public DNS** (or public IP)
- ✅ The username **`Administrator`** and the **decrypted password**
- ✅ A Linux desktop, for example Ubuntu, Debian, Fedora, Linux Mint, or Arch, with a graphical session (X11 or Wayland)
- ✅ `sudo` access to install packages
- ✅ A network that allows **outbound TCP port 3389**. Some campus and office networks block it.

---

## 2. Install an RDP client

Install **Remmina** (GUI), **FreeRDP** (CLI), or both.

### Remmina

| Distribution | Command |
|---|---|
| Ubuntu / Debian / Linux Mint | `sudo apt update && sudo apt install remmina remmina-plugin-rdp` |
| Fedora | `sudo dnf install remmina remmina-plugins-rdp` |
| Arch / Manjaro | `sudo pacman -S remmina freerdp` |
| Any distribution (Flatpak) | `flatpak install flathub org.remmina.Remmina` |

> **Important:** On Debian-based systems, the RDP plugin (`remmina-plugin-rdp`) is a **separate package**. Without it, "RDP" won't appear in Remmina's protocol list.

### FreeRDP (xfreerdp)

| Distribution | Command | Binary name |
|---|---|---|
| Ubuntu 24.04+ / Debian 13+ | `sudo apt install freerdp3-x11` | `xfreerdp3` |
| Older Ubuntu / Debian | `sudo apt install freerdp2-x11` | `xfreerdp` |
| Fedora | `sudo dnf install freerdp` | `xfreerdp` |
| Arch / Manjaro | `sudo pacman -S freerdp` | `xfreerdp3` or `xfreerdp` |

FreeRDP version 3 renamed the binary on some distributions. Check which one you have:

```bash
command -v xfreerdp3 || command -v xfreerdp
```

In this guide, use whichever name the command prints.

---

## 3. Option A: connect with Remmina (GUI)

1. Open **Remmina** from your applications menu.
2. Click the **+** button (top-left) to create a new connection profile.
3. Fill in the **Basic** tab:
   - **Name:** `AWS Windows Server`
   - **Protocol:** **RDP – Remote Desktop Protocol**
   - **Server:** the instance's Public DNS, for example `ec2-13-250-xx-xx.ap-southeast-1.compute.amazonaws.com`
   - **Username:** `Administrator`
   - **Password:** the decrypted password
   - **Domain:** *(leave empty)*
   - **Resolution:** **Use client resolution** or **Dynamic resolution update**
   - **Colour depth:** keep the default. Choose a lower value if the connection is slow.
4. *(Optional)* In the **Advanced** tab, make sure **Security protocol negotiation** is set to **Automatic negotiation**.
5. Click **Save and Connect**.
6. Remmina shows the server's certificate and asks whether to accept it. This is expected. EC2 Windows instances use a self-signed certificate. Click **Yes**.
7. The Windows Server desktop appears. You're connected! 🎉

> **Useful Remmina toolbar buttons**, on the side or top of the session window:
> - **Toggle fullscreen**
> - **Send keystrokes → Ctrl+Alt+Del**
> - **Disconnect**
>
> The default *host key* is **Right Ctrl**. For example, `Right Ctrl + F` toggles fullscreen.

---

## 4. Option B: connect with xfreerdp (CLI)

Run the command below, replacing the host with your instance's Public DNS:

```bash
xfreerdp3 /v:ec2-13-250-xx-xx.ap-southeast-1.compute.amazonaws.com \
  /u:Administrator \
  /cert:ignore \
  /dynamic-resolution \
  +clipboard
```

(Use `xfreerdp` instead of `xfreerdp3` if that's the binary you have.)

FreeRDP prompts for the password. Paste the decrypted password and press **Enter**.

**What the options mean:**

| Option | Meaning |
|---|---|
| `/v:<host>` | The server to connect to |
| `/u:Administrator` | The user name |
| `/cert:ignore` | Accept the instance's self-signed certificate without asking. Fine for a lab, but for anything sensitive, omit it and accept the certificate when prompted. |
| `/dynamic-resolution` | Resize the remote desktop when you resize the window |
| `+clipboard` | Share the clipboard between Linux and Windows |
| `/f` | *(Optional)* Start in fullscreen. Press `Ctrl + Alt + Enter` to leave fullscreen. |

> **Avoid `/p:<password>`.** It works, but the password ends up in your shell history and is visible to other users through `ps`. Let FreeRDP prompt you instead.

---

## 5. After your first login

- **Change the Administrator password.** Inside the remote session, open **PowerShell** and run the command below, then type a new password when prompted. Update the password saved in your Remmina profile afterwards.

  ```powershell
  net user Administrator *
  ```

- **Disconnect vs. sign out:** closing the window *disconnects* you, and your apps keep running on the server. To end the session completely, use **Start → Administrator → Sign out**.

---

## 6. Optional: share folders and the clipboard

### Remmina

1. Right-click your profile and choose **Edit**.
2. In the **Basic** tab, set **Share folder** to a local directory, for example `~/Shared`.
3. In the **Advanced** tab, make sure **Turn off clipboard sync** is **unticked**.
4. Save and reconnect.
5. Inside the instance, open **File Explorer → This PC**. Your folder appears as a network drive.

### xfreerdp

Add a `/drive:` option. The first part is the name shown in Windows, and the second is the local path:

```bash
mkdir -p ~/Shared
xfreerdp3 /v:<Public-DNS-or-IP> /u:Administrator /cert:ignore \
  /dynamic-resolution +clipboard /drive:Shared,$HOME/Shared
```

---

## 7. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Connection timeout, or *"Unable to connect to RDP server"* | The security group doesn't allow your current IP | Update the RDP rule to **My IP**. See [Update your IP in the security group](ec2-setup.md#update-your-ip-in-the-security-group). |
| Timeout even though the security group is correct | The instance is stopped, has no public IP, or your network blocks port 3389 | Check that the instance is **Running** and has a **Public IPv4 address**. Try a phone hotspot to rule out a network block. |
| "RDP" is missing from Remmina's protocol list | The RDP plugin is not installed | `sudo apt install remmina-plugin-rdp` (Debian/Ubuntu), then restart Remmina. |
| `xfreerdp: command not found` | FreeRDP 3 uses a different binary name | Try `xfreerdp3`, or run `command -v xfreerdp3 xfreerdp`. |
| `ERRCONNECT_LOGON_FAILURE` or *"Authentication failure"* | Wrong password or user name | Paste the password again, watching for trailing spaces. Use `Administrator`, and leave **Domain** empty. |
| `ERRCONNECT_TLS_CONNECT_FAILED` or a security negotiation error | Client and server disagree on the security protocol | In Remmina, set **Advanced → Security protocol negotiation** to **NLA**, or to **TLS**. With xfreerdp, add `/sec:nla` or `/sec:tls`. |
| It worked yesterday but not today | The instance was stopped and started, so its public IP changed | Update the Server field with the new Public DNS. To keep a fixed address, attach an [Elastic IP](ec2-setup.md#13-optional-attach-an-elastic-ip). |
| Black screen or very slow desktop | The instance type is too small, or bandwidth is low | Lower the colour depth and resolution. Use `t3.medium` or larger (see [instance types](ec2-setup.md#6-choose-the-instance-type)). |

**Test whether port 3389 is reachable.** Run:

```bash
nc -vz <Public-DNS-or-IP> 3389
```

- `succeeded` or `open` means the network path is open. The problem is your credentials or the client.
- A timeout means something is blocking the connection. Check the security group, the instance state, and your network.

To see detailed connection logs from FreeRDP, add `/log-level:DEBUG` to the command.

For password and key problems, see [EC2 Setup → Troubleshooting](ec2-setup.md#15-troubleshooting-setup-problems).

---

## 8. When you're finished

**Stop** the instance when you aren't using it, and **terminate** it when you're completely done. See [Managing your instance](ec2-setup.md#stop-start-and-terminate) for details.

---

**Other guides:** [EC2 Setup](ec2-setup.md) · [Windows](windows-setup.md) · [macOS](macos-setup.md) · [Main README](../README.md)
