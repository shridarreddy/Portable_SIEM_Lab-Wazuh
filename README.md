# 🛡️ Wazuh-Powered SIEM Lab: Ubuntu Manager + Windows Agents (Portable Setup)

> Build your own cybersecurity lab powered by Wazuh — complete with static IP configuration, custom DNS setup, and dual-adapter virtual networking. Works seamlessly across networks (home, office, offline).

## 📜 Table of Contents
- [Introduction](#introduction)
- [Lab Architecture](#lab-architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Setting up Ubuntu Desktop in VirtualBox](#step-1-setting-up-ubuntu-desktop-in-virtualbox)
- [Step 2: Installing Wazuh All-in-One](#step-2-installing-wazuh-all-in-one)
- [Step 3: Configuring Networking - Bridged + Host-Only](#step-3-configuring-networking---bridged--host-only)
- [Step 4: Assigning Static IP to Ubuntu](#step-4-assigning-static-ip-to-ubuntu)
- [Step 5: Setting up DNS with dnsmasq](#step-5-setting-up-dns-with-dnsmasq)
- [Step 6: Agent Setup (Windows 11 & Server 2022)](#step-6-agent-setup-windows-11--server-2022)
- [Step 7: Confirming Everything Works](#step-7-confirming-everything-works)
- [Troubleshooting Tips](#troubleshooting-tips)
- [Conclusion](#conclusion)

---

## 📘 Introduction

This lab allows you to build a complete SIEM environment using Wazuh as your detection engine and Ubuntu Desktop as your manager.

You’ll learn to:
- Configure virtual machines with custom subnets
- Assign static IPs using ifupdown
- Map DNS names like `wazuh.lab`
- Connect and monitor Windows 11 and Server 2022 machines

---

## 🧱 Lab Architecture

🖥️ The following diagram provides a clear view of the portable SIEM lab setup.

👉 [Click here to view the full architecture diagram](https://wazuh-lab-visual.lovable.app/)

## 📦 Prerequisites

- VirtualBox installed
- Ubuntu Desktop ISO (22.04 or 24.04)
- Windows 11 ISO + Windows Server 2022 ISO (VMs Installed)
- At least 12–16 GB RAM available
- Internet connection during install (optional after setup)

---

## 🧱 Step 1: Setting up **Ubuntu Desktop** in VirtualBox

We start by creating the core system that hosts the Wazuh manager — an **Ubuntu Desktop** VM inside VirtualBox.

### 📦 1.1 Download Ubuntu Desktop ISO

Go to the official Ubuntu site and download the current LTS version:
[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

Choose Ubuntu Desktop 22.04 LTS or 24.04 LTS.

---

### ⚙️ 1.2 Create a New VM in VirtualBox

- Name: Ubuntu Desktop
- Type: Linux
- Version: Ubuntu (64-bit)
- RAM: **8 GB or more**
- CPUs: **2+ recommended**
- Disk: **VHD or VDI**, 60 GB dynamically allocated

---

### 🔌 1.3 Add Network Adapters

We’ll use dual adapters:
- Adapter 1: **Bridged Adapter** → Enables internet access
- Adapter 2: **Host-Only Adapter** → For internal lab traffic

Both should have **“Cable Connected”** enabled in the VirtualBox settings.

![image](https://github.com/user-attachments/assets/20efd856-092a-4b96-9aee-68b5a9d619f7)

---

### 🚀 1.4 Start the VM and Install Ubuntu Normally

Go through the graphical installer and:

- Choose keyboard layout
- Enable LivePatch (optional)
- Create a user
- Enable **OpenSSH server** when prompted
- Set filesystem defaults unless you want custom partitioning

![image](https://github.com/user-attachments/assets/084be09e-75e2-4cda-9e1f-e7db61106f96)

---

### 💡 1.5 Prepare Ubuntu for Wazuh Setup

Once logged in, open Terminal and update all packages:

```bash
sudo apt update && sudo apt upgrade -y
```

This ensures all system components are up to date before we install Wazuh.
---

### 🧰 1.6 (Recommended) Install Guest Additions

This enables:
- Fullscreen support
- Mouse integration
- Clipboard/drag-and-drop

📌 Run this:
```bash
sudo apt install -y virtualbox-guest-utils virtualbox-guest-x11 virtualbox-guest-dkms
```
Then reboot the VM:

```bash
sudo reboot
```

---

## 🔐 Step 2: Installing Wazuh All-in-One

Once your Ubuntu Desktop is ready and Guest Additions are installed, it’s time to set up Wazuh — the heart of your SIEM.

This step installs everything in one go:
- ✅ Wazuh Manager
- ✅ Elasticsearch
- ✅ Wazuh Dashboard
- ✅ Filebeat
- ✅ Wazuh API

---

### 📥 2.1 Download Wazuh Install Script

This command downloads the official Wazuh All-in-One install script:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```
---

### 🔑 2.2 Make It Executable

This sets execution permission on the downloaded script:

```bash
chmod +x wazuh-install.sh
```

---

### 🚀 2.3 Run the Installer (All-in-One)

This command installs the full Wazuh stack:

```bash
sudo ./wazuh-install.sh -a
```
💡 The -a flag stands for All-in-One — installs manager, API, dashboard, and Elasticsearch in the same machine.

![image](https://github.com/user-attachments/assets/afb2ae0c-18d5-4afb-87fa-7b5aa9a9a5f8)

⌛ Installation may take 5–10 minutes.

---

### 🌐 2.4 Access the Wazuh Dashboard

Check the output at the end of the installer — it shows the login URL, user, and auto-generated password.

Open Firefox or your browser (inside VM or host system):

Type: 'https://<your-ip>:443'

![image](https://github.com/user-attachments/assets/7fb6e0e2-cf0f-4603-b870-920f1473fe46)

---

## 🌐 Step 3: Configuring Networking - Bridged + Host-Only

To build a lab network that's both **isolated** and has **internet access**, we’ll use two network adapters:

- **Adapter 1**: Bridged Adapter → Connects your VM to your real network (for internet)
- **Adapter 2**: Host-Only Adapter → Connects lab VMs together with static internal IPs

This setup ensures that communication inside the lab always works, even if you switch Wi-Fi or move between home and office.

---

### 🔧 3.1 Configure Network Adapters in VirtualBox

1. Shut down your **Ubuntu Desktop** VM.
2. Open the VM’s settings → Navigate to the **Network** tab.
3. Configure two adapters:

#### Adapter 1 (Bridged):
- ✅ Enable Adapter
- Attached to: **Bridged Adapter**
- Name: Your current physical internet interface (e.g., Wi-Fi/Ethernet)
- Cable Connected: ✅ Checked

#### Adapter 2 (Host-Only):
- ✅ Enable Adapter
- Attached to: **Host-Only Adapter**
- Name: `vboxnet0`
- Cable Connected: ✅ Checked

---

### 🔍 3.2 Understand How IPs Are Assigned

Once configured:

- **Bridged Adapter** (`enp0s3`) will get its IP from your router (e.g., 192.168.X.X)
- **Host-Only Adapter** (`enp0s8`) will receive a **static IP (like 192.168.56.10)** — you’ll assign this in Step 4

🧠 Host-Only ensures reliable lab communication between VMs, unaffected by external Wi-Fi network changes.

---

### 📌 Why Is Host-Only Important?

If you rely only on bridged mode:
- 🛑 IP will change depending on the current Wi-Fi/internet connection
- ❌ Agents may lose communication to the server

With Host-Only:
- 💡 Traffic stays between VMs, even *offline*
- 🎯 IP remains constant: `192.168.56.10`
- 🧘‍♂️ Portable between locations and future-proof

---

## 🟩 Step 4: Assigning a Static IP to Ubuntu (Host-Only Adapter)

To keep your lab reliable, assign a static IP to the Host-Only adapter (`enp0s8`) on your Ubuntu server.

### 🛠️ How to Set a Static IP (using ifupdown)

1. **Install ifupdown (if not already installed):**
    ```bash
    sudo apt update
    sudo apt install ifupdown -y
    ```

2. **Edit the network interfaces file:**
    ```bash
    sudo nano /etc/network/interfaces
    ```

3. **Add or update these lines:**
    ```
    # Host-only adapter (static lab IP)
    auto enp0s8
    iface enp0s8 inet static
        address 192.168.56.10
        netmask 255.255.255.0
    ```

    *(Make sure `enp0s8` matches your Host-Only adapter’s name. Use `ip a` to check.)*

4. **Restart networking:**
    ```bash
    sudo systemctl restart networking
    ```

5. **Verify the static IP:**
    ```bash
    ip a
    ```
    You should see `192.168.56.10` assigned to `enp0s8`.

> Add screen of `ip a` output showing the static IP on enp0s8

---

Now your Ubuntu server will always have the same IP on the lab network, making DNS and agent connections rock-solid.

![image](https://github.com/user-attachments/assets/5fce9ee0-b888-4877-820d-dcea81fc3681)

---

## 🟦 Step 5: Configuring Private DNS with dnsmasq

Now that your Ubuntu server has a static IP (e.g., `192.168.56.10`), you can set up a private DNS so all your lab machines can use a friendly domain name like `wazuh.lab` instead of typing the IP.

### 🛠️ How Private DNS Works

- Your Ubuntu server runs a lightweight DNS service (`dnsmasq`).
- All lab machines use Ubuntu’s static IP as their DNS server.
- When you type `wazuh.lab`, it automatically resolves to your Wazuh server’s static IP.

### 🛠️ Steps to Install and Configure dnsmasq

1. **Install dnsmasq:**
    ```bash
    sudo apt update
    sudo apt install dnsmasq -y
    ```

2. **Configure your custom lab domain:**
    ```bash
    sudo nano /etc/dnsmasq.conf
    ```
    Add this line at the end:
    ```
    address=/wazuh.lab/192.168.56.10
    ```
    *(Replace `192.168.56.10` with your Ubuntu server’s static IP if different.)*

3. **Restart dnsmasq:**
    ```bash
    sudo systemctl restart dnsmasq
    ```

4. **Set Ubuntu’s IP as DNS on all lab machines:**
    - On Windows: Go to network adapter settings, set **Preferred DNS** to `192.168.56.10` (for Host-Only adapter).
    - On Ubuntu/other Linux VMs: Set DNS to `192.168.56.10` in your network config.

5. **Test it:**
    - On any lab machine, run:
      ```bash
      ping wazuh.lab
      ```
      or open `https://wazuh.lab` in your browser.

![image](https://github.com/user-attachments/assets/4d0f16a8-765d-4267-815d-094124105d12)

---

Now you can use `wazuh.lab` instead of the IP address everywhere in your lab!

![image](https://github.com/user-attachments/assets/2d82199b-d6bc-48c3-bcf4-78aefb27775a)

---

## 🟨 Step 6: Agent Setup (Windows 11 & Windows Server 2022)

Now let’s connect your Windows machines to the Wazuh manager so you can monitor and detect activity across your lab.

### 🖥️ 6.1 Prepare Windows 11 and Windows Server 2022 VMs

- Make sure each Windows VM has:
  - **Adapter 1:** Bridged (for internet, optional)
  - **Adapter 2:** Host-Only (for lab network, required)
- Set the **Preferred DNS** for Adapter 2 to your Ubuntu server’s static IP (e.g., `192.168.56.10`).

> ⚠️ If you can’t ping Ubuntu from Windows, check that Windows Defender Firewall allows ICMP (ping) on the Host-Only network.

![image](https://github.com/user-attachments/assets/05c9ca73-6098-4e8c-8b1f-395577ae6fce)

---

### 🛠️ 6.2 Download and Install the Wazuh Agent

1. **Download the Wazuh agent MSI on each Windows VM:**
    - [Wazuh Agent Download Page](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.3-1.msi)

2. **Run the installer and follow the prompts:**
    - When asked for the Wazuh manager address, enter:
      ```
      wazuh.lab
      ```
      *(Or use the static IP if DNS isn’t working yet)*

---

### 🔑 6.3 Register the Agent with the Manager

1. **On Ubuntu (Wazuh Manager), open a terminal and run:**
    ```bash
    sudo /var/ossec/bin/manage_agents
    ```
2. **Add a new agent:**
    - Press `A` to add
    - Enter a name (e.g., `Windows11` or `Server2022`)
    - Enter the agent’s IP or `any`
    - Save and extract the key

3. **Copy the generated key and paste it into the Wazuh Agent Manager on Windows:**
    - Open the Wazuh Agent Manager (from Start Menu)
    - Paste the key in the Authentication Key field
    - Click Save

4. **Start the agent service:**
    - Open Command Prompt as Administrator and run:
      ```cmd
      net stop wazuh
      net start wazuh
      ```
![image](https://github.com/user-attachments/assets/801d9b07-dfe7-4ef7-ae80-0340318ea488)

---

### 🟢 6.4 Confirm Agent Connection

- Go to the Wazuh Dashboard (`https://wazuh.lab`)
- Navigate to **Agents** and check that your Windows machines show as **Active**

![image](https://github.com/user-attachments/assets/f9a3e4e1-c2eb-4e9d-ac61-67dca4bdf2fb)

---

Now your Windows endpoints are fully monitored by your portable SIEM lab!
---

## 🟩 Step 7: Confirming Everything Works

Now let’s make sure your portable SIEM lab is fully functional and ready for real-world use.

---

### ✅ 7.1 Test DNS Resolution

On any Windows, open a Command Prompt and run:

```bash
ping wazuh.lab
```

You should see replies from your Ubuntu server’s static IP (e.g., `192.168.56.10`).

![image](https://github.com/user-attachments/assets/d6492359-55ca-4b31-8d4e-9f08c2f33216)


---

### ✅ 7.2 Test Wazuh Dashboard Access

Open a browser on any lab machine and go to:

```
https://wazuh.lab
```
or
```
https://192.168.56.10
```

You should see the Wazuh login page. Log in with the credentials provided during installation.

![image](https://github.com/user-attachments/assets/8d8c10f1-cb66-4d30-9b53-9dcaf62a902f)


---

### ✅ 7.3 Confirm Agent Status

- In the Wazuh dashboard, go to **Agents**.
- Both Windows 11 and Windows Server 2022 should show as **Active**.

![image](https://github.com/user-attachments/assets/eeac5343-d402-45df-934b-08845b2f35e7)

---

### ✅ 7.4 Simulate a Test FIM Event (Windows Agent)

To verify File Integrity Monitoring (FIM) is working, create a test file in a directory that is monitored by default.

**Recommended for this lab:**

```cmd
echo wazuh-test > "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\wazuh_test.txt"
```

- Wait 1–2 minutes, then refresh the Wazuh Dashboard.
- Go to **Agents** → select your Windows agent → check the **FIM: Recent events** panel.
- You should see an event for the creation or modification of `wazuh_test.txt`.

> Add a screenshot of the FIM event showing the new file detected

---

![image](https://github.com/user-attachments/assets/4dfb7bad-93f2-4d61-8967-66592e5e84af)

---

If all these checks pass, your portable SIEM lab is fully operational!

---

## 🛠️ Troubleshooting Tips

- **Agent not showing as active?**
  - Double-check the DNS settings on the agent.
  - Make sure the agent is running (`net start wazuh`).
  - Ensure the Ubuntu server’s static IP and DNS are correct.

- **Can’t resolve `wazuh.lab`?**
  - Make sure `dnsmasq` is running on Ubuntu.
  - Verify the `address=/wazuh.lab/192.168.56.10` line in `/etc/dnsmasq.conf`.
  - Check that the agent’s DNS is set to the Ubuntu server’s static IP.

- **Dashboard not loading?**
  - Restart the Wazuh manager and dashboard services:
    ```bash
    sudo systemctl restart wazuh-manager
    sudo systemctl restart wazuh-dashboard
    ```

- **Still stuck?**
  - Reboot all VMs and try again.
  - Review each step above for typos or missed settings.

---

## 🎉 Conclusion

You’ve now built a **portable, real-world SIEM lab** using Wazuh, Ubuntu, and Windows endpoints—all fully virtualized and resilient to network changes.  
This setup is perfect for learning, testing, and demonstrating cybersecurity skills anywhere.

> Add a final screenshot of your full lab dashboard or architecture diagram

---

**Happy hacking and blue teaming!**

---
