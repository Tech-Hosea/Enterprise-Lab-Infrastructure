1️⃣ VLAN Creation

    conf t
    !
    vlan 10
     name HR
    vlan 20
     name IT
    vlan 30
     name FINANCE
    vlan 99
     name MANAGEMENT
    exit
   <img width="308" height="281" alt="switch config" src="https://github.com/user-attachments/assets/435af4d8-3ab9-4064-96e6-3599ad6219ac" />


Explanation:
vlan 10…vlan 99 → creates VLANs with names.
VLAN 10 = HR, VLAN 20 = IT, VLAN 30 = Finance, VLAN 99 = Management.

2️⃣ Trunk to pfSense

    interface GigabitEthernet2/0/2
     description TRUNK_to_pfSense
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,99
     spanning-tree portfast trunk
     no shut
 

Explanation:

This port connects the switch to your pfSense LAN interface.
dot1q → 802.1Q tagging.
switchport mode trunk → allows multiple VLANs over the same port.
allowed vlan 10,20,30,99 → only these VLANs are allowed.
spanning-tree portfast trunk → speeds up STP convergence on trunk port.

3️⃣ Access Ports for HR, IT, Finance

    interface range GigabitEthernet2/0/3-5
     description HR_DESKS
     switchport mode access
     switchport access vlan 10
     spanning-tree portfast
     no shut
    
    interface range GigabitEthernet2/0/6-8
     description IT_DESKS
     switchport mode access
     switchport access vlan 20
     spanning-tree portfast
     no shut
    
    interface range GigabitEthernet2/0/9-11
     description FIN_DESKS
     switchport mode access
     switchport access vlan 30
     spanning-tree portfast
     no shut


Explanation:
Ports 3–5 → HR VLAN 10
Ports 6–8 → IT VLAN 20
Ports 9–11 → Finance VLAN 30
switchport mode access → single VLAN per port.
spanning-tree portfast → skips listening/learning phase for edge devices (PCs).

4️⃣ Switch Management SVI

    no ip routing
    
    interface Vlan99
     description SWITCH_MANAGEMENT
     ip address 192.168.99.2 255.255.255.0
     no shut
    
    ip default-gateway 192.168.99.1
    end
    wr mem


Explanation:
no ip routing → L2 switch mode; no routing between VLANs.
VLAN 99 SVI = 192.168.99.2 → reachable for management.
Default gateway = pfSense 192.168.99.1 → so switch can be managed remotely.
wr mem → saves configuration.

5️⃣ Verification Commands

    show vlan brief
    show interfaces trunk
    show running-config
    ping 192.168.99.1   # pfSense gateway
    ping 192.168.99.100 # Windows Server DC


show vlan brief → check VLANs exist and assigned ports

show interfaces trunk → verify trunk is active

ping → confirm management connectivity
