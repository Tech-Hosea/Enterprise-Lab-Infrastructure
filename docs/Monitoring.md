# Network Monitoring: SNMP on Devices + PRTG Setup
<img width="483" height="314" alt="Screenshot 2025-09-08 124832" src="https://github.com/user-attachments/assets/3cf7feb5-19fd-4089-8849-127cf3a97c95" />

We’ll monitor the **Cisco switch**, **pfSense**, **Windows Server**, and **key VMs**.

---

## A — Enable SNMP on Cisco 3750X (CLI)
<img width="299" height="307" alt="cisco " src="https://github.com/user-attachments/assets/b85fb7f6-216a-4a6e-bbb6-c3ddbeecc5aa" />

On the switch console:

```bash
conf t
 snmp-server community YourCommStr RO
 snmp-server location "Lab Rack"
 snmp-server contact "you@yourdomain.local"
end
wr mem

```

## 🔒 Security note:

    Choose a strong community string instead of public. 
    If possible, prefer SNMPv3 (more complex configuration, but more secure).

## B — Enable SNMP on pfSense (optional)

    Go to Services → SNMP in pfSense.
    Enable SNMP and set a community string (or configure SNMPv3).

  👉 If your version of pfSense does not include SNMP by default, you can:
  
  Monitor pfSense via SSH checks, or
  
  Install an SNMP package if available.

## C — PRTG Installation (High-Level)

PRTG is Windows software.

  Best practice: install on a dedicated monitoring VM.
  
  For a quick lab setup: you can install on your Domain Controller (not recommended long-term).

Steps:
<img width="315" height="236" alt="PRTG" src="https://github.com/user-attachments/assets/82d61a63-2e13-494c-9c22-d0e0e9dd5e2d" />

  Download PRTG from Paessler.
  Run installer → PRTG will create the Probe and web GUI.
  Log into the PRTG web UI (default: http://localhost:8080
  ).

## D — Add Devices & Sensors in PRTG
Cisco Switch

IP: 192.168.10.2 (or mgmt IP, e.g., 192.168.99.2)

  Credentials: SNMP (community = YourCommStr or SNMPv3 credentials).  
  Run Auto-discovery.
  Add useful sensors:

    Interface Traffic    
    Uptime 
    CPU Load   
    Memory Usage

# pfSense

  If SNMP enabled → use SNMP.
  
  If not → use SSH or SNMP via package.
  
  Add sensors:

      Ping
      HTTP
      CPU
      Memory

# Windows Server (Domain Controller)

  * Use WMI / Windows Credentials in PRTG device settings.
  
  * Add sensors:

        CPU
        
        Memory
        
        Disk Free Space
        
        Windows Services (DNS, AD DS, etc.)
        
        Domain Controller Health
✅ Verify that sensors turn green (OK) and that interface traffic shows bits/sec once traffic flows.

## E — Test SNMP

From the PRTG host, run an SNMP test (requires SNMP tools):

      snmpwalk -v2c -c YourCommStr 192.168.99.2


If using SNMPv3:

    snmpwalk -v3 -u prtg -l authPriv -a SHA -A "AuthPass123" -x AES -X "PrivPass123" 192.168.99.2


👉 In PRTG, you can also use the Probe “Test” button in device settings.

## F — Create Maps / Dashboard in PRTG

Go to Maps → Add Map.
Drag device blocks and sensor charts into the map canvas.
Create a topology view:

    pfSense in the center
    Cisco switch
    Domain Controller    
    Client devices (HR, IT, etc.)

This gives you a live network dashboard that updates in real-time.

✅ Outcome

After completing these steps, you will have:
<img width="1396" height="759" alt="Screenshot 2025-09-03 180259" src="https://github.com/user-attachments/assets/f7637158-3ef4-4607-9c89-ec151200a9f4" />

SNMP monitoring on Cisco, pfSense, and Windows Server

PRTG dashboard showing live traffic, uptime, and system health

Alerts and maps for proactive monitoring in your homelab
