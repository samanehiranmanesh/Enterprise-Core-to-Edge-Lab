# Enterprise Core-to-Edge Network Lab — Practical MOP & LinkedIn/GitHub Guide

## 1. Lab Overview & Credentials
- **Topology:** Dual Multilayer Switches (`Multilayer Switch0` and `Multilayer Switch1`) connected across Access switches (`Switch0`–`Switch3`) for VLANs 10, 20, 30, and 99, with `Multilayer Switch1` (`Fa0/5`) routed to `Router0` (`10.0.0.0/30`) and `Loopback0` (`8.8.8.8/32`) simulating WAN reachability.
- **Administrative Credentials:**
  - SSH / Local Username: `netadmin`
  - SSH / Local Password: `NetAdmin123!`
  - Switch Enable Secret: `AdminPass123!`
  - Router Enable Secret: `RouterSec123!`

---

## 2. Step-by-Step Configuration Procedure

### Step 1: Configure `Multilayer Switch0` (HSRP Active, Default Route, DHCP & SSH)
```text
enable
configure terminal
ip routing
interface Vlan10
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
interface Vlan20
 ip address 192.168.20.2 255.255.255.0
 standby 20 ip 192.168.20.1
 standby 20 priority 110
 standby 20 preempt
interface Vlan30
 ip address 192.168.30.2 255.255.255.0
 standby 30 ip 192.168.30.1
 standby 30 priority 110
 standby 30 preempt
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
 standby 99 ip 192.168.99.1
 standby 99 priority 110
 standby 99 preempt
exit
ip route 0.0.0.0 0.0.0.0 192.168.10.3
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp pool VLAN10_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 exit
ip dhcp pool VLAN20_POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 exit
ip dhcp pool VLAN30_POOL
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 exit
service password-encryption
enable secret AdminPass123!
ip domain-name lab.local
crypto key generate rsa general-keys modulus 1024
username netadmin privilege 15 secret NetAdmin123!
line vty 0 15
 transport input ssh
 login local
 exit
banner motd # Authorized Access Only - Network Core #
end
show standby brief
show ip route
write memory
```

### Step 2: Configure `Multilayer Switch1` (HSRP Standby, Routed Port, Default Route & SSH)
```text
enable
configure terminal
ip routing
interface Vlan10
 ip address 192.168.10.3 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 preempt
interface Vlan20
 ip address 192.168.20.3 255.255.255.0
 standby 20 ip 192.168.20.1
 standby 20 preempt
interface Vlan30
 ip address 192.168.30.3 255.255.255.0
 standby 30 ip 192.168.30.1
 standby 30 preempt
interface Vlan99
 ip address 192.168.99.3 255.255.255.0
 standby 99 ip 192.168.99.1
 standby 99 preempt
exit
interface Fa0/5
 no switchport
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.0.0.1
service password-encryption
enable secret AdminPass123!
ip domain-name lab.local
crypto key generate rsa general-keys modulus 1024
username netadmin privilege 15 secret NetAdmin123!
line vty 0 15
 transport input ssh
 login local
 exit
banner motd # Authorized Access Only - Network Core #
end
show standby brief
show ip interface brief
show ip route
write memory
```

### Step 3: Configure `Router0` (Edge Link, Return Static Routes, WAN Loopback & SSH)
```text
enable
configure terminal
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit
ip route 192.168.10.0 255.255.255.0 10.0.0.2
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.99.0 255.255.255.0 10.0.0.2
interface Loopback0
 ip address 8.8.8.8 255.255.255.255
 no shutdown
exit
service password-encryption
enable secret RouterSec123!
ip domain-name lab.local
crypto key generate rsa general-keys modulus 1024
username netadmin privilege 15 secret NetAdmin123!
line vty 0 15
 transport input ssh
 login local
 exit
banner motd # Authorized Access Only - WAN Edge #
end
show ip interface brief
show ip route
ping 10.0.0.2
write memory
```

### Step 4: Verify End-to-End Connectivity on `PC0`
1. Open `PC0` -> `Desktop` -> `IP Configuration` -> select **DHCP** (or confirm IP and default gateway `192.168.10.1`).
2. Open `PC0` -> `Desktop` -> `Command Prompt` and run:
```text
ipconfig
ping 10.0.0.1
ping 8.8.8.8
ping 192.168.20.10
```

---

## 3. Rollback Steps
- If a static route causes an issue, enter `configure terminal` and run `no ip route <prefix> <mask> <next-hop>`.
- If an HSRP or interface setting needs reversal, enter the interface (`interface Vlan10` or `interface Fa0/5`) and negate the specific command (`no standby 10 preempt`, etc.).
- Re-verify with `show ip route`, `show standby brief`, and `show ip interface brief`, then save with `write memory`.

---

## 4. LinkedIn Post Draft (Series Hook for Recruiters & Engineers)
```text
After 10 years in network engineering, one thing is clear: strong cloud and infrastructure engineers are built on rock-solid networking fundamentals.

In Part 1 of this hands-on series, I built and verified a redundant core-to-edge lab in Cisco Packet Tracer:
- Multi-VLAN segmentation (VLANs 10, 20, 30, 99) with HSRP active/standby gateways
- Point-to-point routed edge link (10.0.0.0/30), static routing, and DHCP pools
- Simulated WAN loopback (8.8.8.8) with end-to-end reachability (0% loss)
- SSH hardening, password encryption, local user (netadmin), and MOTD banners

Roadmap from Network Engineering to Cloud:
1. Master switching, routing, subnetting, and packet-level troubleshooting.
2. Lock down HA, SSH/management security, and clean MOP documentation.
3. Build fluency in Linux, virtualization, and containers.
4. Apply core networking to cloud VPC/VNet design, IaC, and automation.

Full MOP and lab notes are in the GitHub repository linked in the comments. Part 2 is next—follow along so you don't miss the next steps in the series.

Where are you focusing right now on your path between core networking and cloud?

#NetworkEngineering #CloudNetworking #Infrastructure #HSRP #SSH #CloudCareer
```

---

## 5. GitHub Repository Setup
Create a repository (for example `network-to-cloud-labs`) with these files:
- `README.md` — Summary of the 10-year network-to-cloud roadmap, Part 1 overview, and link to Part 2.
- `MOP.md` — This practical runbook (`enterprise_core_edge_lab_mop.md`).
- `Senario1.pkt` — Your Packet Tracer topology file.
- `topology.png` and `pc0-ping-test.png` — Your clean screenshots of the topology and `ping` verification.