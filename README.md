# Small Office Network: VLANs, Inter-VLAN Routing and DHCP

A Cisco Packet Tracer lab that simulates a small office network with three departments. Each department is isolated on its own VLAN, gets IP addresses automatically through DHCP, and can reach the other departments through a router.

## Skills demonstrated

- VLAN creation and access port assignment
- 802.1Q trunking
- Inter-VLAN routing (router-on-a-stick)
- DHCP server configuration on a Cisco router
- Basic device hardening (enable secret, disabling unused ports)
- Testing and troubleshooting with `ping`, `show vlan brief` and `ipconfig`

## Tools

- Cisco Packet Tracer
- Cisco 2911 router, Cisco 2960 switch, 6 PCs (simulated)

## Topology

```
             [R1 - Cisco 2911]
                    | G0/0
                    | 802.1Q trunk (VLANs 10, 20, 30)
                    | G0/1
             [SW1 - Cisco 2960]
     /      |      |      |      |      \
  Sales1 Sales2   HR1    HR2    IT1    IT2
  Fa0/1  Fa0/2   Fa0/3  Fa0/4  Fa0/5  Fa0/6
```

## IP addressing plan

| VLAN | Name  | Subnet           | Default gateway | DHCP range                   | Switch ports |
|------|-------|------------------|-----------------|------------------------------|--------------|
| 10   | Sales | 192.168.10.0/24  | 192.168.10.1    | 192.168.10.11 – .254         | Fa0/1–2      |
| 20   | HR    | 192.168.20.0/24  | 192.168.20.1    | 192.168.20.11 – .254         | Fa0/3–4      |
| 30   | IT    | 192.168.30.0/24  | 192.168.30.1    | 192.168.30.11 – .254         | Fa0/5–6      |

Addresses .1–.10 in each subnet are reserved for gateways and future network devices.

## How it works

1. **VLANs** split the single physical switch into three separate networks, so each department's traffic stays isolated by default.
2. **The trunk link** between the switch and router carries all three VLANs over one cable, tagging each frame with its VLAN ID (802.1Q).
3. **Router subinterfaces** (`G0/0.10`, `G0/0.20`, `G0/0.30`) give the router an address in each VLAN. They act as the default gateway and route traffic between departments.
4. **DHCP pools** on the router hand out addresses, the gateway and DNS settings to each PC automatically, based on its VLAN.

## Configuration

### Switch (SW1)

```
hostname SW1
vlan 10
 name Sales
vlan 20
 name HR
vlan 30
 name IT
!
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
interface range fa0/3 - 4
 switchport mode access
 switchport access vlan 20
interface range fa0/5 - 6
 switchport mode access
 switchport access vlan 30
!
interface g0/1
 switchport mode trunk
!
interface range fa0/7 - 24
 shutdown
```

### Router (R1)

```
hostname R1
!
interface g0/0
 no shutdown
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
!
ip dhcp pool SALES
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
ip dhcp pool HR
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
ip dhcp pool IT
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
```

Passwords are removed from the configs in this repository.

## Testing and verification

| Test | Command | Expected result |
|------|---------|-----------------|
| VLAN port assignments | `show vlan brief` (SW1) | Fa0/1–2 in VLAN 10, Fa0/3–4 in VLAN 20, Fa0/5–6 in VLAN 30 |
| Trunk is up | `show interfaces trunk` (SW1) | G0/1 trunking VLANs 10, 20, 30 |
| Subinterfaces are up | `show ip interface brief` (R1) | G0/0.10, .20, .30 up/up |
| DHCP leases | `show ip dhcp binding` (R1) | One lease per PC |
| PC received an address | `ipconfig` (PC) | Address in the correct subnet |
| Inter-VLAN routing | `ping` from Sales1 to HR1 and IT1 | Successful replies |


## Future improvements

- Replace Telnet/console access with SSH
- Enable port security on access ports
- Add a guest Wi-Fi VLAN with an access point
- Add an access control list (ACL) that blocks the guest VLAN from reaching the IT VLAN
- Add a second switch for redundancy


## Author

Noah Hutchinson


