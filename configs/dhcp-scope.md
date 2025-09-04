1️⃣ Lab IP Plan

<img width="663" height="399" alt="DHCP scope" src="https://github.com/user-attachments/assets/c4123057-9b60-495b-8fa4-a8663f393f2f" />
<img width="515" height="154" alt="IP Plan" src="https://github.com/user-attachments/assets/5cefde94-a96b-48e1-9ce1-92b00ba0b3a7" /> 

DNS:
Primary: 192.168.99.100 (Windows Server DC01)
Secondary: same or external 

2️⃣ DHCP Scope for HR VLAN (VLAN 10)

    Scope Name: HR_VLAN10
    IP Range: 192.168.10.50 – 192.168.10.200
    Subnet Mask: 255.255.255.0
    Default Gateway: 192.168.10.1
    Preferred DNS: 192.168.99.100
    Lease Duration: 1 day (for lab)
    Exclusions: 192.168.10.1–192.168.10.49 (reserve static IPs for servers, switches, printers)

Repeat similar steps for IT and Finance VLANs, adjusting subnet and IP ranges.

3️⃣ IT VLAN 20

    Scope Name: IT_VLAN20
    IP Range: 192.168.20.50 – 192.168.20.200
    Gateway: 192.168.20.1
    DNS: 192.168.99.100

4️⃣ Finance VLAN 30 Example

    Scope Name: FINANCE_VLAN30
    IP Range: 192.168.30.50 – 192.168.30.200
    Gateway: 192.168.30.1
    DNS: 192.168.99.100

5️⃣ No DHCP: Management VLAN 99 

    DC01 
    IP: 192.168.99.100
    Gateway: 192.168.99.1
    DNS: 192.168.99.100

6️⃣ How to add these scopes in Windows Server 2022

    Open DHCP console → expand server → IPv4 → Right-click → New Scope.
   
       Follow wizard:
       Name scope (HR_VLAN10, etc.)
       Enter Start/End IP, Subnet mask
       Add exclusions (gateway + static devices)
       Configure Lease duration
       Configure Router (default gateway) → 192.168.x.1
       Configure DNS servers → 192.168.99.100
       Activate scope

Repeat for IT and Finance VLANs.
