# Cisco 3750X VLAN Configuration

## VLAN Definitions
- VLAN 10 – HR (192.168.10.0/24)
- VLAN 20 – IT (192.168.20.0/24)
- VLAN 30 – Finance (192.168.30.0/24)
- VLAN 99 – Management (192.168.99.0/24)

## Example Config
```
conf t
vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name Finance
vlan 99
 name Management
exit

int gi1/0/1
 switchport mode access
 switchport access vlan 10
!
int gi1/0/2
 switchport mode access
 switchport access vlan 20
!
int gi1/0/3
 switchport mode access
 switchport access vlan 30
!
int gi1/0/48
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
```
