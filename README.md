# Active Directory Lab Deployment and Logging Infrastructure Setup

## Introduction  
This project involved the complete setup of a virtualized lab environment to simulate an enterprise Active Directory (AD) network. The lab included domain-joined Windows clients and servers, a Splunk logging server, and Linux-based attack and monitoring platforms. The primary goals of this setup were to:

- Deploy a Windows Server 2022 domain controller and establish an internal domain (`CYBER.LOCAL`)  
- Install and configure Splunk with Universal Forwarders and Sysmon for centralized event logging  
- Set up a Kali Linux machine for adversary emulation  
- Simulate a brute force attack using Hydra and analyze its footprint in Splunk  
- Experiment with Atomic Red Team (ART) techniques to study detection and response  

Through systematic configuration of virtual machines, services, and network connectivity, the following architecture and capabilities were achieved.

---

## 1. Virtual Machine Provisioning and Initial Configuration

### Operating Systems Installed:
- **Windows 10** – Workstation to simulate a domain-joined endpoint  
- **Windows Server 2022** – To act as the AD Domain Controller  
- **Ubuntu (latest)** – Dedicated to running Splunk Enterprise  
- **Kali Linux** – Reserved for offensive security testing  

Each ISO was loaded into VirtualBox and configured with allocated system resources. Hostnames and VM names were assigned for easy identification.

### System Updates and Network Design:
- On **Ubuntu** and **Kali**, the following command was run to update and upgrade packages:  
  ```bash
  sudo apt-get update && sudo apt-get upgrade -y
  ```
- A **NAT Network** was created with the subnet `192.168.10.0/24` and named `ad-project`. All VMs were assigned static IPs within this range and connected to the network.

---

## 2. Splunk Server Setup on Ubuntu

The Ubuntu VM, named `splunk`, was statically assigned `192.168.10.10`. Netplan was configured by editing `/etc/netplan/50-cloud-init.yaml`:
```yaml
dhcp4: no
addresses: [192.168.10.10/24]
nameservers:
  addresses: [8.8.8.8]
routes:
  - to: default
    via: 192.168.10.1
```
Changes were applied via:
```bash
sudo netplan apply
```
Connectivity was verified with `ping 8.8.8.8`.

### Splunk Installation:
- The Splunk `.deb` package was shared via a VirtualBox shared folder.
- Mounting was done with:
  ```bash
  sudo mount -t vboxsf -o uid=1000,gid=1000 Active_Directory share
  ```
- After installing with `sudo dpkg -i splunk.deb`, Splunk was initialized, and a management user was created.
- The service was enabled to start at boot:
  ```bash
  sudo ./splunk enable boot-start -user splunk
  ```

---

## 3. Forwarder Configuration on Windows Machines

### Windows 10 (TARGET-PC):
- IP configured to `192.168.10.100`
- Splunk Universal Forwarder was installed with the receiver set to `192.168.10.10:9997`
- Sysmon was installed with Olaf's config file
- A custom `inputs.conf` was created at `C:\Program Files\SplunkUniversalForwarder\etc\system\local`:
  ```ini
  [WinEventLog://Application]
  index = endpoint
  disabled = false

  [WinEventLog://Security]
  index = endpoint
  disabled = false

  [WinEventLog://System]
  index = endpoint
  disabled = false

  [WinEventLog://Microsoft-Windows-Sysmon/Operational]
  index = endpoint
  disabled = false
  renderXml = true
  ```

- The Splunk Forwarder service was reconfigured to run under the local system account and restarted.

### Windows Server 2022 (ADDS):
- Assigned static IP: `192.168.10.7`
- Same logging and forwarder setup as Windows 10
- Joined to the NAT network and connected successfully to the Splunk instance

---

## 4. Active Directory Deployment

On the Windows Server (ADDS):
- Installed the **Active Directory Domain Services** role
- Promoted the server to a domain controller with the forest name: `CYBER.LOCAL`
- Upon reboot, the domain was fully operational

Two users were created in **Active Directory Users and Computers**, each placed in their respective **Organizational Units (OUs)**.

### Domain Join and Validation:
- The Windows 10 client (`TARGET-PC`) was pointed to the DNS server (`192.168.10.7`)  
- The system was joined to the domain `CYBER.LOCAL` and rebooted  
- Login was tested with one of the newly created AD users

Remote Desktop access was enabled and both users were added to the remote access group.

---

## 5. Offensive Simulation Setup (Planned)

### Kali Linux Configuration:
- Assigned static IP: `192.168.10.250`
- Confirmed connectivity and updated packages

### Planned Hydra Brute Force Simulation:
- Installed with `sudo apt-get install hydra`
- `rockyou.txt` was extracted and the top entries filtered for test:
  ```bash
  head -n 20 rockyou.txt > passwords.txt
  ```
- The plan was to target the Windows domain login and validate brute force detection via Splunk logs

> Due to hardware constraints (8GB RAM), running all VMs simultaneously was not feasible and the brute force test was not fully executed.

---

## 6. Atomic Red Team (Planned Tests)

The setup included plans to install and run Atomic Red Team on the Windows client:
- **T1136.001 – Create Local Accounts**
- **T1059.001 – PowerShell Execution**

These would be used to observe and study detections via Sysmon logs forwarded to Splunk.

---

## Conclusion

This project successfully established a multi-VM Active Directory lab environment integrated with centralized logging using Splunk and Sysmon. While some offensive simulations were constrained due to resource limitations, the core infrastructure was completed and verified.

**Key Components:**

| Component | Hostname | IP Address | Role |
|----------|----------|------------|------|
| Ubuntu   | splunk   | 192.168.10.10 | Splunk Enterprise Server |
| Windows 10 | TARGET-PC | 192.168.10.100 | Domain-joined client |
| Windows Server | ADDS | 192.168.10.7 | Domain Controller |
| Kali Linux | kali | 192.168.10.250 | Attack simulation (Hydra) |

**Highlights:**
- Functional AD environment with working DNS and domain login
- Centralized logging from clients and domain controller
- Custom Splunk inputs for granular event visibility
- Prepared structure for red team simulation and detection tuning

This lab serves as a foundational environment for future adversary emulation, detection engineering, and defensive testing.

---

