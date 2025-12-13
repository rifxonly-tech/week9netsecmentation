# Network Segmentation & Firewall Implementation

## Overview

This project demonstrates a **secure network segmentation design** using **Cisco Packet Tracer**, based on the Network Segmentation case study provided in the course PDF (pages 37–39).

The implementation follows best practices by separating **User Network**, **DMZ (Server Network)**, and **External Network (Internet Simulation)** using routers and an **ASA 5506-X firewall**. All configurations were tested and verified using end-to-end connectivity checks.

---

## Network Topology Summary

### Network Segments

| Segment               | Network          | Purpose                        |
| --------------------- | ---------------- | ------------------------------ |
| User LAN              | 192.168.100.0/24 | Internal user PCs              |
| Inside Transit        | 10.10.10.0/24    | Router1 ↔ ASA                  |
| Outside Transit       | 20.20.20.0/24    | ASA ↔ Router2                  |
| DMZ / Server LAN      | 172.16.10.0/24   | Application & Database Servers |
| Internet (Simulation) | 8.8.8.0/24       | External network simulation    |

### Devices Used

* Router-PT (Router1)
* Router-PT (Router2)
* ASA 5506-X (ASA0)
* Switch 2960 (User, Server, Internet)
* PC-PT (User PCs)
* Server-PT (Application / Database / Internet)

---

## IP Addressing Plan

### Router1

* Fa0/0 (to User Switch): `192.168.100.1/24`
* Fa1/0 (to ASA inside): `10.10.10.2/24`

### ASA Firewall

* Gi1/1 (inside): `10.10.10.1/24`
* Gi1/2 (outside): `20.20.20.1/24`
* Gi1/3 (dmz): `172.16.10.1/24`

### Router2

* Fa1/0 (to ASA): `20.20.20.2/24`
* Fa0/0 (to Internet Switch): `8.8.8.1/24`

### Servers

* DMZ Servers: `172.16.10.11 – 172.16.10.14`
* Internet Server: `8.8.8.8`

---

## CLI Configuration (FINAL & VERIFIED)

### Router1 Configuration

```bash
enable
configure terminal

interface fastEthernet0/0
 ip address 192.168.100.1 255.255.255.0
 no shutdown
 exit

interface fastEthernet1/0
 ip address 10.10.10.2 255.255.255.0
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 10.10.10.1

end
write
```

---

### Router2 Configuration

```bash
enable
configure terminal

interface fastEthernet1/0
 ip address 20.20.20.2 255.255.255.0
 no shutdown
 exit

interface fastEthernet0/0
 ip address 8.8.8.1 255.255.255.0
 no shutdown
 exit

ip route 192.168.100.0 255.255.255.0 20.20.20.1
ip route 10.10.10.0 255.255.255.0 20.20.20.1
ip route 172.16.10.0 255.255.255.0 20.20.20.1

end
write
```

---

### ASA 5506-X Firewall Configuration

#### Interface Configuration

```bash
enable
configure terminal

interface gigabitEthernet1/1
 nameif inside
 security-level 100
 ip address 10.10.10.1 255.255.255.0
 no shutdown
 exit

interface gigabitEthernet1/2
 nameif outside
 security-level 0
 ip address 20.20.20.1 255.255.255.0
 no shutdown
 exit

interface gigabitEthernet1/3
 nameif dmz
 security-level 50
 ip address 172.16.10.1 255.255.255.0
 no shutdown
 exit
```

#### Default Route

```bash
route outside 0.0.0.0 0.0.0.0 20.20.20.2
```

#### NAT Configuration

```bash
object network USER_LAN
 subnet 192.168.100.0 255.255.255.0
 nat (inside,outside) dynamic interface

object network SERVER_LAN
 subnet 172.16.10.0 255.255.255.0
 nat (dmz,outside) dynamic interface
```

#### Basic Firewall Policy

```bash
access-list OUTSIDE_IN extended deny ip any 192.168.100.0 255.255.255.0
access-list OUTSIDE_IN extended deny ip any 172.16.10.0 255.255.255.0
access-group OUTSIDE_IN in interface outside

end
write memory
```

---

## Testing & Validation

### Connectivity Tests (From User PC)

```bash
ping 192.168.100.1   # Router1
ping 10.10.10.1      # ASA inside
ping 20.20.20.2      # Router2
ping 8.8.8.8         # Internet server
```

Expected result: **All pings successful**

### Security Test (From Internet Server)

```bash
ping 192.168.100.10
```

Expected result: **Request timeout (blocked by firewall)**

---

## Conclusion

This implementation successfully fulfills all requirements of the Network Segmentation assignment:

* Network segmentation using multiple subnets
* Firewall segmentation using ASA 5506-X
* Proper routing and NAT configuration
* Secure separation between internal users, servers, and external network

The final topology is fully operational, secure, and aligned with the provided case study.

---

**Status:** ✅ Completed and Verified
