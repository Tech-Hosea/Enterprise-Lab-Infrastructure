# ) Enable OpenVPN (Remote Access to Simulate Remote Work)

## 🎯 Objective
The objective of this lab is to configure **secure remote access** using pfSense OpenVPN.  
- Remote employees should be able to connect from outside the network and access **HR resources**.  
- VPN clients must **not** access **IT resources** unless explicitly allowed.  
- The setup enforces **network segmentation, access control, and DNS integration** with Active Directory to simulate a real-world enterprise remote work environment.

---

## A — Install Client Export Package (One-Time)

1. Navigate to:  
   `System → Package Manager → Available Packages`
2. Install **openvpn-client-export**.  
   - After install, a new menu appears under `VPN`.

---

## B — Use the OpenVPN Wizard (Wizard → OpenVPN)

1. Go to:  
   `VPN → OpenVPN → Wizards → Remote Access (User Auth)`
2. Setup:
   - **CA**: Create new CA (if none exists).
   - **Server Cert**: Create certificate for server.
   - **Authentication**: Local Database *(or Local+RADIUS if available; use Local DB for lab)*.
   - **Server Interface**: `WAN`
   - **Protocol**: `UDP (recommended)`
   - **Local Port**: `1194`
   - **Tunnel Network**: `10.8.0.0/24` (VPN clients get `10.8.0.x`)
   - **Local Network(s)**: `192.168.10.0/24,192.168.99.0/24`  
     *(HR + MGMT networks; if UI only accepts one, add routes manually in advanced config).*
   - **Crypto Settings**: Defaults are fine (`AES-256`, `SHA256`).
3. Finish wizard → creates CA, server cert, server instance, and OpenVPN firewall rule.  
   *(We’ll refine rules below).*

---

## C — Add OpenVPN Firewall Rules

Go to:  
`Firewall → Rules → OpenVPN`

- **Block OpenVPN → IT**
  - Action: `Block`
  - Protocol: `Any`
  - Source: `OpenVPN net`
  - Destination: `192.168.20.0/24`
  - Log: Enabled

- **Allow OpenVPN → HR (VLAN10)**
  - Action: `Pass`
  - Protocol: `Any` *(or limit to required ports)*
  - Source: `OpenVPN net`
  - Destination: `192.168.10.0/24`
  - Description: `VPN -> HR resources`

- **Allow OpenVPN → MGMT (optional, restricted)**
  - Allow only specific source user/group or target host/ports (e.g., RDP to jumpbox).

- **Allow OpenVPN → Any (optional)**
  - If you want split tunnel or full internet via VPN.  
  - Ensure NAT handles VPN traffic (`Firewall → NAT → Outbound`).  
    pfSense usually adds this automatically.

---

## D — Create Users & Export Client Config

1. Go to:  
   `System → User Manager → Add`  
   Create users for VPN access (username/password).
2. Go to:  
   `VPN → OpenVPN → Client Export`
3. Select user → export installer or `.ovpn` file.
4. Install OpenVPN client on remote laptop.  
   Import `.ovpn` and connect.

---

## E — Test VPN Connection

From remote device (offsite):

1. Connect via OpenVPN client.
2. Verify:
   - `ipconfig` / `ifconfig` → should show `10.8.0.x`.
   - `ping 192.168.10.10` (HR machine) → **Success**.
   - `ping 192.168.20.5` (IT resource) → **Blocked**.
   - `nslookup someinternalhost` → should resolve via AD DNS.  
     *(Push DNS option in OpenVPN advanced settings if needed).*

**To push DNS & routes:**
push "dhcp-option DNS 192.168.99.100"
push "route 192.168.10.0 255.255.255.0"

---

# 5) Test Client Isolation — Verification

### From HR Client (VLAN10):
- `ping 192.168.10.1` (gateway) → ✅ success  
- `ping 192.168.99.100` (DC) → ✅ success  
- `ping 192.168.20.5` (IT) → ❌ blocked  
- Check pfSense firewall logs → confirm block entry.

### From IT Client (VLAN20):
- `ping 192.168.20.1` → ✅ success  
- `ping 192.168.10.50` → ❌ blocked  

### From VPN Client:
- `ping 192.168.10.50` → ✅ allowed  
- `ping 192.168.20.5` → ❌ blocked  
- `nslookup dcname` → ✅ resolves to `192.168.99.100`  
- AD join operations & Group Policy updates → ✅ should work.

---
