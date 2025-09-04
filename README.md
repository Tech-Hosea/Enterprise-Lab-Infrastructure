# 🏢 Enterprise IT Infrastructure Homelab

This project simulates a **real-world enterprise IT environment** using a Dell PowerEdge T330, Windows Server 2022, pfSense firewall, and Cisco Catalyst 3750X switch.  
It demonstrates skills in **Active Directory, Group Policy, VLAN segmentation, firewall rules, VPN, and SNMP monitoring** — core responsibilities for IT Support, SysAdmin, and Cybersecurity roles.

---

## 🚀 Lab Overview

- **Active Directory (AD DS)** – with OUs for HR, IT, Finance
- **Group Policy** – password policies, drive mapping, USB restrictions
- **pfSense Firewall** – WAN/LAN config, firewall rules, VPN setup
- **Cisco 3750X** – VLANs, trunks, access ports
- **DNS/DHCP** – domain-integrated DNS, DHCP scopes & reservations
- **File Sharing** – NTFS permissions by department
- **Monitoring** – SNMP monitoring with PRTG and network topology map

---

## 🛠️ Technologies Used
- **Windows Server 2022** – AD DS, DNS, DHCP, GPO
- **pfSense** – Firewall, VPN (OpenVPN), DHCP Relay
- **Cisco Catalyst 3750X** – VLANs, trunking
- **PRTG / LibreNMS** – SNMP monitoring
- **Windows 10 Clients** – domain-joined workstations

---

## 📌 Key Features

### 🔹 Active Directory & Group Policy
- OUs: HR, IT, Finance  
- Users & groups with naming convention: `firstname.lastname`  
- GPOs applied:  
  - Strong password policy  
  - Drive mappings (H:, F:)  
  - Disable USB storage devices  
  - OU-specific restrictions  

### 🔹 pfSense Firewall
- WAN: `192.168.1.183` (DHCP from ISP modem)  
- LAN: `192.168.10.1/24` (static)  
- Rules:  
  - Allow HR → Internet  
  - Block IT → HR subnet  
  - VPN enabled for remote access (OpenVPN)  

### 🔹 Cisco 3750X VLANs
- VLAN 10 – HR  
- VLAN 20 – IT  
- VLAN 30 – Finance  
- VLAN 99 – Management  
- Trunk between pfSense & switch  
- Access ports mapped to departments  

### 🔹 File Sharing & NTFS Permissions
- HR share → Modify for HR, Read-only for others  
- Finance share → Finance-only access  
- Verified access control using domain logins  

### 🔹 Network Monitoring
- SNMP enabled on pfSense, Cisco switch, and Windows Server  
- Monitored in PRTG dashboard  
- Auto-generated **topology map**  

---

## 📊 Network Diagram

![Lab Diagram](./docs/lab-diagram.png)

---

## 🎯 Skills Demonstrated
✔️ IT Infrastructure design & deployment  
✔️ Identity & Access Management  
✔️ VLAN segmentation & firewall rules  
✔️ VPN configuration & testing  
✔️ File system permissions & access control  
✔️ SNMP monitoring & alerting  
✔️ Troubleshooting & documentation  

---

## ✅ Next Steps
- Add **WSUS server** for patch management  
- Deploy **SIEM (Wazuh/Graylog)** for log analysis  
- Configure **site-to-site VPN** for branch simulation  

---

👨‍💻 *Built in my Dell PowerEdge T330 Homelab to simulate enterprise IT environments for IT Support, System Administration, and Cybersecurity readiness.*
