# pfSense Firewall Rules

## LAN (192.168.10.0/24)
- Allow HR VLAN to Internet
- Block IT VLAN to HR VLAN

## Management VLAN (192.168.99.0/24)
- Allow access to pfSense web interface
- Allow SNMP queries

## VPN
- Allow OpenVPN clients to access internal resources
