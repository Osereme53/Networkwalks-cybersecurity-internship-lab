## Cybersecurity Testing Lab Setup (WK1-PM1)

A step-by-step record of building a cybersecurity testing lab on a Windows laptop using **VirtualBox** and **Kali Linux** as the attacking machine.

## Task Requirements

![Task WK1-PM1](images/IMG_6538.jpeg)

- Use VirtualBox as the base (latest recommended version)
- Set up Kali Linux as the attacking/hacker machine
- Use a NAT Network with subnet `10.0.0.0/24`
- Enable clipboard and file drag/drop
- Enable a shared folder (host `Downloads` folder)
- Kali IP address must be `10.0.0.2/24`
- Kali must have full Internet access

## Lab Summary

| Item | Value |
|---|---|
| Hypervisor | Oracle VirtualBox 7.2.6 (amd64) |
| Attacker VM | Kali Linux (`Attacking/hacker machine`) |
| RAM / CPU | 4096 MB / 2 processors |
| Disk | 40 GB VDI (dynamically allocated) |
| Network | NAT Network `NatNetwork`, `10.0.0.0/24`, DHCP enabled |
| Kali IP | `10.0.0.2/24` (static) |
| Gateway / DNS | `10.0.0.1` / `8.8.8.8` |
| Shared folder | Host `Downloads` mounted at `/mnt/downloads` |

---

## Step 1: Install VirtualBox

Download the latest VirtualBox installer for Windows and run it. This lab used **version 7.2.6 (amd64)**. Click **Next** through the wizard and finish the installation.

![VirtualBox installer](images/IMG_6594.png)

## Step 2: Create the Kali Linux VM

Create a new VM named **Attacking/hacker machine** in VirtualBox Manager:

- Type/OS: Linux (64-bit)
- Base memory: 4096 MB
- Processors: 2
- Hard disk: 40 GB VDI, dynamically allocated

![VirtualBox Manager with VM details](images/IMG_6604.jpeg)

Storage configuration (Kali disk attached to SATA Port 0, optical drive empty after installation):

![VM storage settings](images/IMG_6606.jpeg)

## Step 3: Create the NAT Network (10.0.0.0/24)

In VirtualBox go to **File > Tools > Network Manager > NAT Networks** and create a network:

- Name: `NatNetwork`
- IPv4 Prefix: `10.0.0.0/24`
- Enable DHCP: checked
- Enable IPv6: unchecked

![NAT Network configuration](images/IMG_6605.jpeg)

Then open the VM **Settings > Network**, set **Adapter 1** to **NAT Network** and choose `NatNetwork`.

## Step 4: Enable Clipboard and Drag & Drop

Open VM **Settings > General > Features** (Expert mode) and set:

- Shared Clipboard: **Bidirectional**
- Drag-and-Drop: **Bidirectional**

![Clipboard and drag/drop](images/IMG_6612.jpeg)

## Step 5: Configure the Shared Folder

Open VM **Settings > Shared Folders** and add a machine folder:

- Folder Path: the host `Downloads` folder
- Access: **Full**
- Auto Mount: **Yes**

![Shared folder settings](images/IMG_6613.jpeg)

## Step 6: Set the Static IP on Kali (10.0.0.2/24)

By default, DHCP assigned Kali a different address (`10.0.2.5/24`):

![Default DHCP address](images/IMG_6608.jpeg)

Open **Edit Connections > Wired connection 1 > IPv4 Settings** and set:

| Field | Value |
|---|---|
| Method | Manual |
| Address | `10.0.0.2` |
| Netmask | `24` |
| Gateway | `10.0.0.1` |
| DNS servers | `8.8.8.8` |

Click **Save**.

![IPv4 manual settings](images/IMG_6607.jpeg)

## Step 7: Apply the Changes

Bring the interface down and up. Note that `ip ifconfig` is not a valid command, so use `ifconfig` or `ip link`:

```bash
sudo ifconfig eth0 down
sudo ifconfig eth0 up
# or
sudo ip link set eth0 up
```

Then restart NetworkManager (mind the spelling):

```bash
sudo systemctl restart NetworkManager
```

![Restarting NetworkManager](images/IMG_6609.jpeg)

> **Troubleshooting notes**
> - `Object "ifconfig" is unknown` happens because `ifconfig` is not an `ip` subcommand. Use `ifconfig eth0 ...` or `ip link set eth0 ...`.
> - `Unit NetworkManage.service not found` was a typo. The correct service name is `NetworkManager`.

## Step 8: Verify the Network Configuration

```bash
nmcli device status
ip -br addr
ip route
```

Expected results:

- `eth0` is **connected** using "Wired connection 1"
- `eth0` shows `10.0.0.2/24`
- Default route goes via `10.0.0.1`

![Verifying IP and routes](images/IMG_6610.jpeg)

## Step 9: Verify Internet Access

```bash
ping -c 4 google.com
```

Result: 4 packets transmitted, 4 received, **0% packet loss**, so Kali has full Internet access.

![Ping test](images/IMG_6611.jpeg)

## Step 10: Verify the Shared Folder

Inside Kali, the shared folder is mounted at `/mnt/downloads`, owned by group `vboxsf`:

```bash
ls -la /mnt/downloads
touch /mnt/downloads/kali_shared_test.txt
ls -la /mnt/downloads
```

The test file `kali_shared_test.txt` was created and should also appear in the Windows `Downloads` folder, confirming read/write access.

> If you get a permission error as the `kali` user, add it to the `vboxsf` group and log out/in:
> ```bash
> sudo usermod -aG vboxsf $USER
> ```
> A `No such file or directory` error was caused by a typo (`/mnt/dowloads`).

![Shared folder test](images/IMG_6614.jpeg)

## Step 11 (Bonus): Vulnerability Scanner Setup (GVM/OpenVAS)

Greenbone Vulnerability Management was installed and checked with:

```bash
sudo gvm-check-setup
```

Result: **"It seems like your GVM-25.04.0 installation is OK."** The `gvmd` and `gsad` services are active, and `nmap`, `ssh-keygen`, `nsis`, and `xsltproc` were found.

Warnings seen (non-blocking):
- `GLib-CRITICAL ... g_file_get_contents` assertion message, which is cosmetic
- Password policy is empty. Optionally edit `/etc/gvm/pwpolicy.conf`

![GVM check setup](images/IMG_6615.jpeg)

---

## Requirements Checklist

- [x] VirtualBox as base (v7.2.6)
- [x] Kali Linux as attacking machine
- [x] NAT Network on `10.0.0.0/24`
- [x] Clipboard and drag/drop enabled (bidirectional)
- [x] Shared folder with host Downloads
- [x] Kali IP is `10.0.0.2/24`
- [x] Full Internet access (ping test passed)

## Repository Structure

```
.
├── README.md
└── images/
    ├── IMG_6538.jpeg
    ├── IMG_6594.png
    └── ... (all screenshots)
```

> **Ethical note:** This lab is for learning and authorized testing only. Only test systems you own or have written permission to test.
