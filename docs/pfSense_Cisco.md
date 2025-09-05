# pfSense + Cisco VLAN Lab

## Objective
Configure pfSense as the firewall and router for the lab, integrate it with a Cisco Catalyst switch, and implement VLAN trunking, firewall rules, and client isolation for a enterprise setup.

---

## 0) Lab Prerequisites
- **Hardware/VMs:**
  - Dell PowerEdge T330 (running pfSense VM, Windows Server, Windows 10 VM)
  - Cisco Catalyst 3750X switch
- **Network Plan:**
  - pfSense WAN: `DHCP` (from ISP or upstream router)
  - pfSense LAN (Mgmt / VLAN 99): `192.168.99.1/24`
  - Cisco Switch SVI VLAN 99: `192.168.99.2/24`
  - Windows Server (DC01): `192.168.99.100/24`
  - Windows 10 Client: `192.168.99.110/24`
- **Console Access:** pfSense WebGUI + Cisco switch console (USB/Serial).
<img width="1165" height="526" alt="pfSense " src="https://github.com/user-attachments/assets/2e97045f-2899-4adb-a972-0a856ae0cc98" />

---

## 1) pfSense Initial Setup
1. Assign interfaces:
   - WAN → `em0` (DHCP from ISP )
   - LAN → `em1` → set to `192.168.99.1/24`
2. Enable WebGUI:  
   - Browse to `https://192.168.99.1` from Windows client.  
   - Login with default creds `admin/pfsense`.
3. Disable default anti-lockout rule (optional after rules are configured).

---

## 2) pfSense Firewall Rules
1. Go to **Firewall → Rules → LAN**:
   - Allow all traffic from `LAN net` → `any` (temporary).
   - Later tighten to specific VLANs.
2. Create rule for **Mgmt VLAN (99)**:
   - Source: `192.168.99.0/24`
   - Destination: `any`
   - Action: Pass

---

## 3) Cisco Switch VLAN Trunking
On Catalyst 3750X:
<img width="488" height="284" alt="ciscotrunk" src="https://github.com/user-attachments/assets/53babcf7-1ec9-47b0-937e-8d54d1f0b721" />

```cisco
conf t
! Define VLANs
vlan 10
 name Workstations
vlan 20
 name Servers
vlan 99
 name Management
exit

! Assign management IP to VLAN 99
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown

! Trunk uplink to pfSense
interface gi2/0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 spanning-tree portfast trunk
```

## 4) pfSense VLAN Interfaces
<img width="468" height="273" alt="pfSense Vlan" src="https://github.com/user-attachments/assets/a3db77cf-dc7e-40e7-a7ae-536517048f73" />

   
    Navigate: Interfaces → Assignments → VLANs
    
    Add VLAN 10 (Workstations) → Parent: LAN (em1), Tag: 10
    
    Add VLAN 20 (Servers) → Parent: LAN (em1), Tag: 20
    
    Assign new interfaces:
    
    OPT1 → VLAN 10 → IP 192.168.10.1/24
    
    OPT2 → VLAN 20 → IP 192.168.20.1/24
    
    Enable DHCP servers:
    
    VLAN 10: Range 192.168.10.50 - 192.168.10.200
    
    VLAN 20: Range 192.168.20.50 - 192.168.20.200
    
## 5) VLAN Client Isolation
  <img width="527" height="458" alt="pfSensefirewallrule" src="https://github.com/user-attachments/assets/c57aff46-843f-42d2-b511-6de59efeded3" />

    In Firewall → Rules:
    
    Block traffic between VLANs (default deny).
    
    Example rule to allow only Mgmt VLAN (99) to access pfSense:
    
    Source: 192.168.99.0/24
    
    Destination: pfSense
    
    Action: Pass
    
    Block all inter-VLAN traffic except what’s explicitly allowed.
    
    Test by:
    
    Ping between VLAN 10 and VLAN 20 clients → should fail.
    
    Ping VLAN 99 to VLAN 10 → should work if rules allow.

## 6) Verification & Testing

pfSense Diagnostics → ARP Table
Confirm switch + clients show correct MAC/IP mappings.

    Cisco CLI
    
    show vlan brief
    show ip int brief
    show arp


Client Testing

Windows 10 in VLAN 10 gets DHCP from 192.168.10.1.

Server in VLAN 20 gets DHCP from 192.168.20.1.

VLAN isolation works as per rules.
