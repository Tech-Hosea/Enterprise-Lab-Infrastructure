# Active Directory Lab Setup (DC01)

This guide walks through building a Windows Server 2022 Domain Controller (DC01) with **AD DS, DNS, DHCP, OUs, Users, Groups, File Shares, Group Policy, and Domain-Joined Clients**.  
It’s structured step-by-step so you can reproduce it in a homelab or training environment.

---

## 0) Prereqs & One-Time Prep

### Set static IP (DC01)
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

4. Dynamic DNS updates:  
   - DHCP → Properties → DNS tab → **Always dynamically update DNS A and PTR records**  
   - Enable cleanup when lease is deleted.  
   - (Optional) Configure DNS credentials.

5. Reservations:  
   - Example: `WIN10-01`, IP `192.168.10.61`, MAC `AA-BB-CC-DD-EE-FF`.

---

## 4) Build OU Structure

ADUC → right-click domain → New → Organizational Unit:


