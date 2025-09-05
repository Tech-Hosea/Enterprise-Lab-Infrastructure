# Active Directory Lab Setup (DC01)

This guide walks through building a Windows Server 2022 Domain Controller (DC01) with **AD DS, DNS, DHCP, OUs, Users, Groups, File Shares, Group Policy, and Domain-Joined Clients**.  
It’s structured step-by-step so you can reproduce it in a homelab or training environment.

---

## 0) Prereqs & One-Time Prep

### Set static IP (DC01)
<img width="421" height="238" alt="staticIP(DC01)" src="https://github.com/user-attachments/assets/e34235d4-3eb1-4f42-9579-4183aa9bb1c2" />

Server Manager → Local Server → Ethernet → Properties → IPv4:
- **IP**: `192.168.99.100`  
- **Mask**: `255.255.255.0`  
- **Gateway**: `192.168.99.1`  
- **DNS**: `192.168.99.100` (itself)

### Rename the server
- Server Manager → Local Server → Computer name → **Change…** → restart.  
- New name: **DC01**

### Time source
- pfSense should have accurate time (NTP).  
- Windows syncs from the domain later.

### DHCP conflict check
- Ensure no other DHCP server is active on the subnet.

---

## 1) Install AD DS & Promote DC (New Forest)
<img width="443" height="309" alt="installAD" src="https://github.com/user-attachments/assets/8d97babd-8109-431f-9fc2-b57c8856243b" />

1. Add the role:  
   Server Manager → Add roles and features → Role-based → select **DC01**  
   → Check **Active Directory Domain Services** (accept DNS) → Install.

2. Promote to domain controller:  
   Server Manager (flag in top right) → **Promote this server to a domain controller**  
   → Add a new forest → Root domain: `teech-hosea.lab`  
   → Set **DSRM password** → Install (auto reboot).

3. Verify AD/DNS after reboot:  
   - Tools → **Active Directory Users and Computers (ADUC)**  
   - Tools → **DNS** → confirm `tech-hosea.local` forward lookup zone.

---

## 2) Configure DNS (Forwarders & Reverse Zone)
<img width="387" height="239" alt="forwarder Reversezone" src="https://github.com/user-attachments/assets/d2de99d6-2b10-42fd-94bc-a6fefc244ade" />

- **Forwarders**:  
  DNS → right-click server → Properties → Forwarders tab → Edit → add `192.168.99.1` (pfSense).  

- **Reverse lookup zone**:  
  DNS → Reverse Lookup Zones → New Zone…  
  - Primary zone  
  - IPv4 Reverse Lookup  
  - Network ID: `192.168.99`  

---

## 3) Install & Configure DHCP on DC01

1. Add DHCP role:  
   Server Manager → Add roles and features → **DHCP Server** → Install.

2. Post-install task:  
   Server Manager (flag) → Complete DHCP configuration → **Authorize** with domain creds.

3. Create a scope:
   
    <img width="563" height="299" alt="DHCP scope" src="https://github.com/user-attachments/assets/ea01c703-c2c6-4d14-9967-44468292658f" />

   - Name: `LAN-192.168.10.0/24`  (HR)                         
   - Range: `192.168.10.50–192.168.10.200`                                                                         
   - Router: `192.168.99.1`  
   - DNS: `192.168.99.100` (DC01)  
   - Domain: `tech-hosea.local`
   - Activate scope.

   - Name: `LAN-192.168.20.0/24`  (IT)
   - Range: `192.168.20.50–192.168.10.200`  
   - Router: `192.168.20.1`  
   - DNS: `192.168.99.1` (DC01)  
   - Domain: `tech-hosea.local`  
   - Activate scope.

   - Name: `LAN-192.168.30.0/24`  (Finance)
   - Range: `192.168.30.50–192.168.10.200`  
   - Router: `192.168.30.1`  
   - DNS: `192.168.99.1` (DC01)  
   - Domain: `tech-hosea.local`  
   - Activate scope.
  
   - Activate scope.
   - Name: `LAN-192.168.20.0/24`  (IT)
   - Range: `192.168.20.50–192.168.10.200`  
   - Router: `192.168.20.1`  
   - DNS: `192.168.99.1` (DC01)  
   - Domain: `tech-hosea.local`  
   - Activate scope.

5. Dynamic DNS updates:  
   - DHCP → Properties → DNS tab → **Always dynamically update DNS A and PTR records**  
   - Enable cleanup when lease is deleted.  
   - (Optional) Configure DNS credentials.

6. Reservations:  
   - Example: `WIN10-01`, IP `192.168.10.61`, MAC `AA-BB-CC-DD-EE-FF`.

---

## 4) Build OU Structure

ADUC → right-click domain → New → Organizational Unit:

      Corp
      ├── Users
      │ ├── HR
      │ ├── IT
      │ └── Finance
      ├── Groups
      ├── Workstations
      └── Servers
      

---

## 5) Create Users & Groups (AGDLP Model)
<img width="332" height="199" alt="OU" src="https://github.com/user-attachments/assets/58c1186a-6322-4ec9-9b95-b2cea19cfaea" />

**Naming conventions**:  
- Users: `jdoe`, `asmith`  
- Groups:  
  - Global: `GG_HR_Users`, `GG_IT_Users`, `GG_Finance_Users`  
  - Domain Local: `DL_FS_HR_RW`, `DL_FS_HR_RO`, `DL_FS_Finance_RW`, `DL_FS_Finance_RO`

**Example users**:  
- John Doe (`jdoe`) → member of `GG_HR_Users`  
- Alice Smith (`asmith`) → member of `GG_Finance_Users`  
- IT Admin (`itadmin`) → member of `GG_IT_Users` + `Domain Admins`

---

## 6) File Server Shares & NTFS Permissions
<img width="380" height="258" alt="FileShare" src="https://github.com/user-attachments/assets/b35bcc5f-9738-4e43-ba11-f7cf16788c89" />

**Create folders** (preferably on a dedicated file server):  

D:\Shares\HR
D:\Shares\Finance
D:\Shares\Public


**Shares**:  
- HR → `HR$`  
- Finance → `Finance$`
<img width="342" height="259" alt="Fileshare02" src="https://github.com/user-attachments/assets/a04a8eac-fa19-4a55-b464-bcf8b2e0db76" />

**NTFS Permissions**:  
- `DL_FS_HR_RW` → Modify  
- `DL_FS_HR_RO` → Read  
- Similar for Finance.

**Group nesting (AGDLP)**:  
Users → Global Groups → Domain Local → Permissions.

---

## 7) Group Policy (GPOs)

### 7A) Domain password policy
Default Domain Policy →  
<img width="518" height="293" alt="Policy" src="https://github.com/user-attachments/assets/fefcb81d-dd42-437b-bb87-a146d8818ffb" />

- Min length: 12  
- History: 24  
- Complexity: Enabled  
- Max age: 90 days  
- Lockout: 5 attempts / 15 minutes.

### 7B) Disable USB storage
New GPO → Link to **Workstations OU**:  
`Computer Config → Policies → Admin Templates → System → Removable Storage Access → Deny all access`

### 7C) Map drives (per dept.)
New GPO → User Config → Preferences → Windows Settings → Drive Maps  
<img width="374" height="268" alt="Drivemap" src="https://github.com/user-attachments/assets/cd4c0832-08bc-4d8c-9f0c-250789da9f88" />

- HR Share (H:) → target `DL_FS_HR_RW`/`RO`  
- Finance Share (F:) → target `DL_FS_Finance_RW`/`RO`

---

## 8) Join a Windows 10 Client
<img width="476" height="333" alt="Users" src="https://github.com/user-attachments/assets/1d86057f-49ac-4ab3-888a-99821de09d55" />

1. Set DNS → `192.168.99.1`  
2. Renew IP (`ipconfig /release /renew`)  
3. Join domain: `corp.lab` → provide Domain Admin creds → reboot.  
4. Move into **Workstations OU**.

---

## 9) Verify GPOs Apply

On CLIENT01:  

```powershell
gpupdate /force
gpresult /r



