# PROJECT 1: Enterprise Network Infrastructure Modernization & Active Directory Consolidation

**Company:** Infogain India Private Limited, Bangalore  
**Role:** Infrastructure Engineer - Network & Systems Support Lead  
**Duration:** January 2019 - June 2023 (4.5 years)  
**Project Timeline:** June 2021 - December 2022 (18 months)  
**Budget:** ₹12,500,000 (~$150,000 USD)  
**Team Size:** 8 members

---

## Executive Summary

Led comprehensive network infrastructure modernization project consolidating three fragmented networks from company acquisitions into unified, secure, and monitored infrastructure. Replaced 42 end-of-life switches with standardized Cisco equipment, migrated 850 users from three Active Directory domains to single domain, redesigned IP addressing scheme with VLAN segmentation, and deployed centralized monitoring (PRTG) with 576 sensors.

**Key Results:**
- Eliminated 47 monthly IP conflicts (100% reduction)
- Improved network uptime from 97.2% to 99.6%
- Reduced help desk tickets by 68% (180→58 per month)
- Achieved 99% success rate for AD migration (850 users, 840 computers)
- ROI: ₹13.9M annual savings, 10.8-month payback period

---

## Business Challenge

### Initial Environment (2021)

**Three Separate Networks (Acquisition Legacy):**
```
Network 1: 172.16.0.0/16 (Building A - HQ, 400 users)
├── Cisco 2960 switches (12 units, 8 years old)
├── Windows Server 2008 R2 Domain Controllers
└── Domain: infogain-hq.local

Network 2: 10.10.0.0/16 (Building B - Development, 300 users)
├── Cisco 2950 switches (8 units, 11 years old)
├── Windows Server 2012 Domain Controller
└── Domain: infogain-dev.local

Network 3: 192.168.0.0/16 (Buildings C & D - Support/Ops, 250 users)
├── D-Link mixed models (6 units, 5-9 years old)
├── Windows Server 2008 Domain Controller
└── Domain: infogain-support.local
```

**Critical Problems Documented:**
- 47 IP address conflicts per month
- 15-20 DNS resolution failures per week
- 180 network-related help desk tickets monthly
- 2-3 switch failures per month (end-of-life equipment)
- No centralized monitoring or network documentation
- Three isolated Active Directory domains (no trusts)
- Inconsistent security policies across networks
- No configuration backups (manual only)

---

## Technical Implementation

### Phase 1: Network Design & Architecture (Jun-Aug 2021)

#### IP Addressing Scheme (10.50.0.0/16)

```
┌─────────────────────────────────────────────────────────────┐
│ VLAN 10: Management (10.50.0.0/24)                          │
│ Purpose: Switch/router management interfaces                 │
│ Gateway: 10.50.0.1                                          │
│ Devices: 42 switches, 2 core switches, 1 firewall          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 20: Servers (10.50.10.0/24)                           │
│ Purpose: Domain controllers, DHCP, DNS, file servers        │
│ Gateway: 10.50.10.1                                         │
│ Key IPs: DC01 (.10), DC02 (.11), DHCP (.21), Files (.20)   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 30-33: Employee Workstations                          │
├─────────────────────────────────────────────────────────────┤
│ VLAN 30: Building A (10.50.20.0/23) - 510 IPs             │
│ VLAN 31: Building B (10.50.22.0/23) - 510 IPs             │
│ VLAN 32: Building C (10.50.24.0/24) - 254 IPs             │
│ VLAN 33: Building D (10.50.25.0/24) - 254 IPs             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 40: Printers (10.50.30.0/24)                          │
│ Purpose: Network printers with DHCP reservations            │
│ Gateway: 10.50.30.1                                         │
│ Static Reservations: 25 printers (10.50.30.10-34)          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 50: VoIP (10.50.40.0/24)                              │
│ Purpose: IP phones with QoS priority                        │
│ Gateway: 10.50.40.1                                         │
│ DHCP Option 150: TFTP Server 10.50.10.25                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 60: Guest WiFi (10.50.60.0/24)                        │
│ Purpose: Guest network isolated via ACL                     │
│ Gateway: 10.50.60.1                                         │
│ Security: Blocked from internal networks (10.50.0.0/16)     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ VLAN 70: IoT Devices (10.50.70.0/24)                       │
│ Purpose: Cameras, access control, HVAC                      │
│ Gateway: 10.50.70.1                                         │
│ Security: Blocked from internet access                      │
└─────────────────────────────────────────────────────────────┘
```

**Design Rationale:**
- 20-30% IP headroom per subnet for growth
- Consistent third octet (20s=employees, 30s=printers, 40s=VoIP)
- Security segmentation by function and location
- Easy identification for troubleshooting

#### Network Topology

```
                         Internet (200 Mbps)
                                |
                         [ISP Router]
                         203.0.113.1
                                |
                    [FortiGate 100F Firewall]
                    WAN: 203.0.113.5/29
                    LAN: 10.50.0.254/24
                                |
                    ┌───────────┴───────────┐
                    │   Core Switch Stack    │
                    │  (2x Catalyst 3850)    │
                    │  VLAN Routing (SVIs)   │
                    │  OSPF Area 0           │
                    │  10.50.0.1             │
                    └───────────┬───────────┘
                                |
        ┌───────────┬───────────┼───────────┬───────────┐
        |           |           |           |           |
   [Dist-A]    [Dist-B]    [Dist-C]    [Dist-D]   [Data Center]
   Bldg A      Bldg B      Bldg C      Bldg D      Servers
   400 users   300 users   150 users   100 users   DC01, DC02
        |           |           |           |       DHCP, DNS
   [12 Access] [10 Access] [8 Access]  [6 Access]
   [Switches]  [Switches]  [Switches]  [Switches]
```

#### Equipment Selection

| Device Type | Model | Quantity | Purpose | Cost (₹) |
|-------------|-------|----------|---------|----------|
| Core Switch | Cisco Catalyst 3850-48P | 2 | Stacked, Layer 3 routing | 3,200,000 |
| Distribution | Cisco Catalyst 2960X-48FPS | 8 | Building distribution (2/bldg) | 4,800,000 |
| Access | Cisco Catalyst 2960X-24PS | 32 | Floor-level access | 3,600,000 |
| Firewall | Fortinet FortiGate 100F | 1 | Internet edge, NAT, IPS | 450,000 |
| Monitoring Server | Windows Server 2019 | 1 | PRTG Network Monitor | 150,000 |
| **Total** | | **44** | | **₹12,200,000** |

---

### Phase 2: Physical Installation (Nov 2021 - Jan 2022)

#### Phased Rollout Schedule

```
Week 1 (Nov 6-7): Building D - PILOT
├── Users: 100 (smallest, least critical)
├── Equipment: 6 access switches
├── Downtime: Saturday 8 AM - 6 PM
├── Pre-work: Labeled 87 cables, port mapping spreadsheet
├── Result: ✅ Zero extended downtime
└── Lessons: Cable labeling saved 30 min/switch

Week 2 (Nov 13-14): Building C
├── Users: 150
├── Equipment: 8 access switches
├── Downtime: Saturday 8 AM - 6 PM
├── Issue: 1 legacy printer needed static IP (resolved)
└── Result: ✅ Completed on schedule

Week 3 (Nov 19-21): Building B - EXTENDED WINDOW
├── Users: 300
├── Equipment: 10 access switches
├── Downtime: Friday 10 PM - Sunday 6 PM
├── Issues: 
│   ├── VLAN mismatch on trunk (corrected)
│   ├── Faulty SFP module (replaced from spare)
│   └── Spanning tree loop (15 min outage - see incident below)
└── Result: ✅ Completed with lessons learned

Week 4 (Nov 26-28): Building A - CRITICAL (HQ)
├── Users: 400
├── Equipment: 12 access switches
├── Downtime: Friday 10 PM - Sunday 6 PM
├── Issues:
│   ├── Old patch panel wiring errors (re-terminated 5 cables)
│   └── VoIP VLAN missing on 2 switches (corrected)
└── Result: ✅ All users online by 5 PM Sunday

Week 5 (Dec 4): Core Switch Cutover - CRITICAL
├── Impact: ALL 850 USERS
├── Equipment: 2x Catalyst 3850 (stacked)
├── Downtime: Sunday 6 AM - 6 PM
├── Team: 5 engineers on-site
└── Result: ✅ Completed in 11 hours (1 hour early)
```

#### Core Switch Installation Timeline

```
06:00 AM ┃ Team arrival, safety briefing, final checklist review
06:30 AM ┃ Graceful shutdown of old core switch
07:00 AM ┃ Physical installation begins
         ┃ ├── Rack mounting (2x 3850 switches)
         ┃ ├── StackWise-480 cable connection
         ┃ ├── Dual power supplies to separate UPS circuits
         ┃ └── 8 fiber uplinks (2 per building)
         ┃
09:00 AM ┃ Power on, stack formation
09:15 AM ┃ Stack formed: Switch 1 = Master (priority 15)
09:30 AM ┃ Apply pre-staged configuration via console
         ┃
10:00 AM ┃ Verify Layer 3 configuration
         ┃ ├── show ip interface brief (all SVIs up)
         ┃ ├── show vlan brief (all VLANs active)
         ┃ └── show spanning-tree summary (root bridge confirmed)
         ┃
10:30 AM ┃ Configure OSPF routing
         ┃ ├── router ospf 1
         ┃ ├── router-id 10.50.0.1
         ┃ └── network 10.50.0.0 0.0.255.255 area 0
         ┃
11:00 AM ┃ Configure DHCP relay (ip helper-address 10.50.10.21)
11:30 AM ┃ Test DHCP from each VLAN (test laptops)
         ┃
12:00 PM ┃ LUNCH BREAK (monitoring dashboards)
         ┃
12:30 PM ┃ Bring up building links incrementally:
         ┃ ├── 12:30 - Building D link up → test connectivity ✓
         ┃ ├── 13:00 - Building C link up → test connectivity ✓
         ┃ ├── 13:30 - Building B link up → test connectivity ✓
         ┃ └── 14:00 - Building A link up → test connectivity ✓
         ┃
14:30 PM ┃ All buildings operational
15:00 PM ┃ Performance testing
         ┃ ├── Inter-VLAN routing (Finance → Servers)
         ┃ ├── Internet connectivity (all VLANs)
         ┃ └── Load test (200 simultaneous connections)
         ┃
16:00 PM ┃ Validation complete
16:30 PM ┃ Documentation and handoff
17:00 PM ┃ Email notification: "Network maintenance complete"
         ┃ ALL 850 USERS BACK ONLINE ✅
```

---

### Phase 3: Layer 2/3 Configuration (Nov 2021 - Jan 2022)

#### Core Switch Configuration

```cisco
!========================================
! CORE SWITCH STACK CONFIGURATION
!========================================
! Stack configuration
switch 1 priority 15
switch 2 priority 14

! Hostname
hostname CORE-STACK-DC

!========================================
! VLAN CREATION
!========================================
vlan 10
 name MANAGEMENT
vlan 20
 name SERVERS
vlan 30
 name EMPLOYEES-A
vlan 31
 name EMPLOYEES-B
vlan 32
 name EMPLOYEES-C
vlan 33
 name EMPLOYEES-D
vlan 40
 name PRINTERS
vlan 50
 name VOIP
vlan 60
 name GUEST-WIFI
vlan 70
 name IOT-DEVICES

!========================================
! LAYER 3 SVIs (INTER-VLAN ROUTING)
!========================================
interface vlan 10
 description MANAGEMENT-VLAN
 ip address 10.50.0.1 255.255.255.0
 no shutdown

interface vlan 20
 description SERVERS-VLAN
 ip address 10.50.10.1 255.255.255.0
 no shutdown

interface vlan 30
 description EMPLOYEES-BUILDING-A
 ip address 10.50.20.1 255.255.254.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 31
 description EMPLOYEES-BUILDING-B
 ip address 10.50.22.1 255.255.254.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 32
 description EMPLOYEES-BUILDING-C
 ip address 10.50.24.1 255.255.255.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 33
 description EMPLOYEES-BUILDING-D
 ip address 10.50.25.1 255.255.255.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 40
 description PRINTERS-VLAN
 ip address 10.50.30.1 255.255.255.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 50
 description VOIP-VLAN
 ip address 10.50.40.1 255.255.255.0
 ip helper-address 10.50.10.21
 no shutdown

interface vlan 60
 description GUEST-WIFI
 ip address 10.50.60.1 255.255.255.0
 ip helper-address 10.50.10.21
 ip access-group GUEST-ISOLATION in
 no shutdown

interface vlan 70
 description IOT-DEVICES
 ip address 10.50.70.1 255.255.255.0
 no shutdown

!========================================
! OSPF ROUTING CONFIGURATION
!========================================
router ospf 1
 router-id 10.50.0.1
 network 10.50.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface GigabitEthernet1/0/1
 no passive-interface GigabitEthernet1/0/2

!========================================
! DEFAULT ROUTE TO FIREWALL
!========================================
ip route 0.0.0.0 0.0.0.0 10.50.0.254

!========================================
! SPANNING TREE (ROOT BRIDGE)
!========================================
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,31,32,33,40,50,60,70 priority 4096

!========================================
! TRUNK PORTS TO DISTRIBUTION SWITCHES
!========================================
interface range GigabitEthernet1/0/1-8
 description UPLINKS-TO-BUILDINGS
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,31,32,33,40,50,60,70
 spanning-tree portfast trunk
```

#### Access Switch Standard Configuration Template

```cisco
!========================================
! STANDARD ACCESS SWITCH CONFIGURATION
!========================================
! Example: Building A, Floor 2, IDF 1

!========================================
! BASIC CONFIGURATION
!========================================
hostname SWITCH-A-F2-IDF1
ip domain-name infogain.local

!========================================
! MANAGEMENT VLAN
!========================================
vlan 10
 name MANAGEMENT

interface vlan 10
 ip address 10.50.0.25 255.255.255.0
 no shutdown

ip default-gateway 10.50.0.1

!========================================
! SECURITY - PASSWORDS
!========================================
enable secret 5 $1$mERr$hAVpqG8H2zK8T.mQyPqM01
service password-encryption

username admin privilege 15 secret AdminP@ss2021!

!========================================
! CONSOLE & VTY ACCESS
!========================================
line console 0
 password 7 08314D5D1A0E1B
 login
 logging synchronous
 exec-timeout 15 0

line vty 0 15
 transport input ssh
 login local
 access-class MGMT-ACCESS in
 exec-timeout 30 0

!========================================
! SSH CONFIGURATION
!========================================
crypto key generate rsa modulus 2048
ip ssh version 2

! Disable Telnet
no ip telnet server

!========================================
! SNMP CONFIGURATION (v3 Secure)
!========================================
snmp-server group MONITORING v3 priv
snmp-server user monitoradmin MONITORING v3 auth sha AuthP@ss123 priv aes 128 PrivP@ss123
snmp-server host 10.50.0.50 version 3 priv monitoradmin
snmp-server location "Building A - Floor 2 - IDF 1"
snmp-server contact "IT Support - support@infogain.com"
snmp-server enable traps

!========================================
! NTP CONFIGURATION
!========================================
ntp server 10.50.10.10 prefer
ntp server 10.50.10.11

!========================================
! SYSLOG CONFIGURATION
!========================================
logging host 10.50.0.50
logging trap informational
logging source-interface vlan 10

!========================================
! VLANS
!========================================
vlan 20
 name SERVERS
vlan 30
 name EMPLOYEES-A
vlan 40
 name PRINTERS
vlan 50
 name VOIP

!========================================
! TRUNK PORT (UPLINK)
!========================================
interface GigabitEthernet0/1
 description UPLINK-TO-DIST-SWITCH-A
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
 spanning-tree portfast trunk

!========================================
! ACCESS PORTS (WORKSTATIONS)
!========================================
interface range GigabitEthernet0/2-12
 description EMPLOYEE-WORKSTATIONS
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 spanning-tree bpduguard enable
 !
 ! PORT SECURITY
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security aging time 10
 switchport port-security aging type inactivity

!========================================
! VOIP PORTS (WORKSTATION + PHONE)
!========================================
interface range GigabitEthernet0/13-24
 description EMPLOYEE-WORKSTATIONS-WITH-VOIP
 switchport mode access
 switchport access vlan 30
 switchport voice vlan 50
 spanning-tree portfast
 spanning-tree bpduguard enable
 !
 ! PORT SECURITY
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky

!========================================
! DISABLE UNUSED PORTS
!========================================
interface range GigabitEthernet0/25-48
 shutdown

!========================================
! ACCESS CONTROL LIST (MANAGEMENT)
!========================================
ip access-list standard MGMT-ACCESS
 10 permit 10.50.0.0 0.0.0.255
 20 permit 10.50.10.0 0.0.0.255
 99 deny any log

!========================================
! DHCP SNOOPING
!========================================
ip dhcp snooping
ip dhcp snooping vlan 30,40,50

! Trust uplink only
interface GigabitEthernet0/1
 ip dhcp snooping trust

! Rate limit access ports
interface range GigabitEthernet0/2-24
 ip dhcp snooping limit rate 10

!========================================
! DYNAMIC ARP INSPECTION
!========================================
ip arp inspection vlan 30,40,50

! Trust uplink
interface GigabitEthernet0/1
 ip arp inspection trust

! Rate limit access ports
interface range GigabitEthernet0/2-24
 ip arp inspection limit rate 15

!========================================
! STORM CONTROL
!========================================
interface range GigabitEthernet0/2-24
 storm-control broadcast level 10.00
 storm-control multicast level 20.00
 storm-control action shutdown
 storm-control action trap

!========================================
! DISABLE UNNECESSARY SERVICES
!========================================
no cdp run
no ip http server
ip http secure-server
no service finger
no service tcp-small-servers
no service udp-small-servers
no ip source-route
service tcp-keepalives-in
service tcp-keepalives-out

!========================================
! LOGGING & AUDITING
!========================================
archive
 log config
  logging enable
  logging size 1000
  notify syslog contenttype plaintext

!========================================
! BANNER
!========================================
banner motd ^
*************************************************************
AUTHORIZED ACCESS ONLY - Infogain India Pvt Ltd
All activities are monitored and logged
Unauthorized access will be prosecuted
*************************************************************
^

!========================================
! SAVE CONFIGURATION
!========================================
end
copy running-config startup-config
```

#### ACL Configuration Examples

```cisco
!========================================
! GUEST WIFI ISOLATION ACL
!========================================
ip access-list extended GUEST-ISOLATION
 10 remark ALLOW-GUEST-TO-INTERNET
 10 permit ip 10.50.60.0 0.0.0.255 any
 20 remark DENY-GUEST-TO-INTERNAL-NETWORKS
 20 deny ip 10.50.60.0 0.0.0.255 10.50.0.0 0.0.255.255 log
 99 remark ALLOW-ALL-OTHER-TRAFFIC
 99 permit ip any any

! Apply to guest VLAN interface
interface vlan 60
 ip access-group GUEST-ISOLATION in

!========================================
! IOT DEVICE SECURITY ACL
!========================================
ip access-list extended IOT-SECURITY
 10 remark DENY-IOT-TO-INTERNET
 10 deny ip 10.50.70.0 0.0.0.255 any log
 20 remark ALLOW-IOT-TO-MANAGEMENT-VLAN
 20 permit ip 10.50.70.0 0.0.0.255 10.50.0.0 0.0.0.255
 99 remark DENY-ALL-ELSE
 99 deny ip any any log

! Apply to IoT VLAN interface
interface vlan 70
 ip access-group IOT-SECURITY in
```

---

### Phase 4: Firewall Configuration (Dec 2021)

#### FortiGate 100F Configuration

```bash
#========================================
# FORTINET FORTIGATE 100F CONFIGURATION
#========================================

# System settings
config system global
    set hostname "FGT-EDGE-01"
    set timezone "Asia/Kolkata"
end

#========================================
# INTERFACE CONFIGURATION
#========================================
config system interface
    edit "wan1"
        set mode static
        set ip 203.0.113.5 255.255.255.248
        set allowaccess ping https ssh
        set description "ISP-CONNECTION"
    next
    edit "internal"
        set mode static
        set ip 10.50.0.254 255.255.255.0
        set allowaccess ping https ssh
        set vdom "root"
        set description "INTERNAL-NETWORK"
    next
end

#========================================
# STATIC ROUTING
#========================================
config router static
    edit 1
        set gateway 203.0.113.1
        set device "wan1"
        set comment "DEFAULT-ROUTE-TO-ISP"
    next
    edit 2
        set dst 10.50.0.0 255.255.0.0
        set gateway 10.50.0.1
        set device "internal"
        set comment "ROUTE-TO-CORE-SWITCH"
    next
end

#========================================
# FIREWALL ADDRESS OBJECTS
#========================================
config firewall address
    edit "RFC1918-10.50.0.0/16"
        set subnet 10.50.0.0 255.255.0.0
        set comment "INTERNAL-NETWORK"
    next
    edit "EMPLOYEE-VLANS"
        set subnet 10.50.20.0 255.255.248.0
        set comment "BUILDINGS-A-B-C-D"
    next
    edit "GUEST-WIFI"
        set subnet 10.50.60.0 255.255.255.0
        set comment "GUEST-NETWORK"
    next
    edit "IOT-DEVICES"
        set subnet 10.50.70.0 255.255.255.0
        set comment "CAMERAS-ACCESS-CONTROL"
    next
end

#========================================
# FIREWALL POLICIES
#========================================
config firewall policy
    edit 1
        set name "INTERNAL-TO-INTERNET"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "EMPLOYEE-VLANS"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS" "NTP"
        set nat enable
        set logtraffic all
        set comments "EMPLOYEES-INTERNET-ACCESS"
    next
    edit 2
        set name "GUEST-TO-INTERNET-ONLY"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "GUEST-WIFI"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set nat enable
        set logtraffic all
        set comments "GUEST-INTERNET-ACCESS"
    next
    edit 3
        set name "BLOCK-GUEST-TO-INTERNAL"
        set srcintf "internal"
        set dstintf "internal"
        set srcaddr "GUEST-WIFI"
        set dstaddr "RFC1918-10.50.0.0/16"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set comments "ISOLATE-GUEST-NETWORK"
    next
    edit 4
        set name "BLOCK-IOT-TO-INTERNET"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "IOT-DEVICES"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set comments "IOT-SECURITY-POLICY"
    next
    edit 5
        set name "SERVERS-TO-INTERNET"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "10.50.10.0/24"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTPS" "DNS" "NTP"
        set nat enable
        set logtraffic all
        set comments "SERVER-UPDATE-ACCESS"
    next
end

#========================================
# INTRUSION PREVENTION (IPS)
#========================================
config ips sensor
    edit "default"
        set comment "PROTECT-ALL-TRAFFIC"
        config entries
            edit 1
                set severity critical high medium
                set action block
                set log enable
            next
        end
    next
end

# Apply IPS to firewall policies
config firewall policy
    edit 1
        set ips-sensor "default"
    next
    edit 2
        set ips-sensor "default"
    next
    edit 5
        set ips-sensor "default"
    next
end

#========================================
# GEOBLOCKING (HIGH-RISK COUNTRIES)
#========================================
config firewall address
    edit "GEO-BLOCKED-COUNTRIES"
        set type geography
        set country "CN" "RU" "KP" "IR"
        set comment "HIGH-RISK-COUNTRIES"
    next
end

config firewall policy
    edit 99
        set name "BLOCK-HIGH-RISK-COUNTRIES"
        set srcintf "wan1"
        set dstintf "internal"
        set srcaddr "GEO-BLOCKED-COUNTRIES"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end

#========================================
# NAT/PAT CONFIGURATION
#========================================
# All internal traffic uses single public IP (PAT)
# Configured via "nat enable" in firewall policies above
# Session table supports up to 500,000 concurrent connections

#========================================
# LOGGING
#========================================
config log syslogd setting
    set status enable
    set server "10.50.0.50"
    set port 514
    set facility local7
end

config log fortianalyzer setting
    set status disable
end
```

**Firewall Policy Summary:**

| Policy | Source | Destination | Action | NAT | Purpose |
|--------|--------|-------------|--------|-----|---------|
| 1 | Employees | Internet | Allow | Yes | Business internet access |
| 2 | Guest WiFi | Internet | Allow | Yes | Guest internet only |
| 3 | Guest WiFi | Internal | **Deny** | No | **Isolate guests** |
| 4 | IoT Devices | Internet | **Deny** | No | **Prevent IoT compromise** |
| 5 | Servers | Internet | Allow | Yes | Updates and patches |
| 99 | Blocked Countries | Internal | **Deny** | No | **Geo-blocking** |

---

### Phase 5: DNS/DHCP Infrastructure (Oct-Nov 2021)

#### DNS Server Configuration

```powershell
#========================================
# WINDOWS DNS SERVER CONFIGURATION
#========================================
# Primary DNS: DC01 (10.50.10.10)
# Secondary DNS: DC02 (10.50.10.11)

# DNS Zones
New-DnsServerPrimaryZone -Name "infogain.local" `
    -ReplicationScope "Forest" `
    -DynamicUpdate "Secure"

New-DnsServerPrimaryZone -NetworkId "10.50.0.0/16" `
    -ReplicationScope "Forest" `
    -DynamicUpdate "Secure"

# DNS Forwarders (External resolution)
Add-DnsServerForwarder -IPAddress "8.8.8.8"
Add-DnsServerForwarder -IPAddress "8.8.4.4"

# Conditional Forwarders (Migration period only)
Add-DnsServerConditionalForwarderZone -Name "infogain-hq.local" `
    -MasterServers "172.16.10.10"
Add-DnsServerConditionalForwarderZone -Name "infogain-dev.local" `
    -MasterServers "10.10.20.10"
Add-DnsServerConditionalForwarderZone -Name "infogain-support.local" `
    -MasterServers "192.168.10.10"

# DNS Scavenging (Stale record cleanup)
Set-DnsServerScavenging -ScavengingState $true `
    -RefreshInterval 7.00:00:00 `
    -NoRefreshInterval 7.00:00:00 `
    -ScavengingInterval 7.00:00:00

Set-DnsServerZoneAging -Name "infogain.local" `
    -Aging $true `
    -RefreshInterval 7.00:00:00 `
    -NoRefreshInterval 7.00:00:00

# Key DNS Records
Add-DnsServerResourceRecordA -Name "dc01" -ZoneName "infogain.local" `
    -IPv4Address "10.50.10.10"
Add-DnsServerResourceRecordA -Name "dc02" -ZoneName "infogain.local" `
    -IPv4Address "10.50.10.11"
Add-DnsServerResourceRecordA -Name "dhcp" -ZoneName "infogain.local" `
    -IPv4Address "10.50.10.21"
Add-DnsServerResourceRecordA -Name "fileserver" -ZoneName "infogain.local" `
    -IPv4Address "10.50.10.20"

# DNS Performance: Average query response 12ms (down from 45ms)
```

#### DHCP Server Configuration

```powershell
#========================================
# WINDOWS DHCP SERVER CONFIGURATION
#========================================
# DHCP Server: Windows Server 2019 (10.50.10.21)

# Install and authorize DHCP server
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName "dhcp.infogain.local" `
    -IPAddress "10.50.10.21"

#========================================
# BUILDING A EMPLOYEES
#========================================
Add-DhcpServerv4Scope -Name "Building-A-Employees" `
    -StartRange 10.50.20.100 -EndRange 10.50.21.254 `
    -SubnetMask 255.255.254.0 `
    -State Active `
    -LeaseDuration 8.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.20.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -DnsDomain "infogain.local" `
    -Router 10.50.20.1

#========================================
# BUILDING B EMPLOYEES
#========================================
Add-DhcpServerv4Scope -Name "Building-B-Employees" `
    -StartRange 10.50.22.100 -EndRange 10.50.23.254 `
    -SubnetMask 255.255.254.0 `
    -State Active `
    -LeaseDuration 8.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.22.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -DnsDomain "infogain.local" `
    -Router 10.50.22.1

#========================================
# BUILDING C EMPLOYEES
#========================================
Add-DhcpServerv4Scope -Name "Building-C-Employees" `
    -StartRange 10.50.24.100 -EndRange 10.50.24.254 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 8.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.24.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -DnsDomain "infogain.local" `
    -Router 10.50.24.1

#========================================
# BUILDING D EMPLOYEES
#========================================
Add-DhcpServerv4Scope -Name "Building-D-Employees" `
    -StartRange 10.50.25.100 -EndRange 10.50.25.254 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 8.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.25.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -DnsDomain "infogain.local" `
    -Router 10.50.25.1

#========================================
# PRINTERS (WITH RESERVATIONS)
#========================================
Add-DhcpServerv4Scope -Name "Printers" `
    -StartRange 10.50.30.50 -EndRange 10.50.30.200 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 30.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.30.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -DnsDomain "infogain.local" `
    -Router 10.50.30.1

# Sample printer reservations (25 total created)
Add-DhcpServerv4Reservation -ScopeId 10.50.30.0 `
    -IPAddress 10.50.30.10 -ClientId "00-11-22-33-44-55" `
    -Description "Building-A-Floor1-Printer-HP-LaserJet"

Add-DhcpServerv4Reservation -ScopeId 10.50.30.0 `
    -IPAddress 10.50.30.11 -ClientId "00-11-22-33-44-66" `
    -Description "Building-A-Floor2-Printer-Canon-MF445"

#========================================
# VOIP PHONES
#========================================
Add-DhcpServerv4Scope -Name "VOIP-Phones" `
    -StartRange 10.50.40.100 -EndRange 10.50.40.250 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 30.00:00:00

Set-DhcpServerv4OptionValue -ScopeId 10.50.40.0 `
    -DnsServer 10.50.10.10,10.50.10.11 `
    -Router 10.50.40.1 `
    -OptionId 150 -Value "10.50.10.25"  # TFTP server for phone configs

#========================================
# GUEST WIFI
#========================================
Add-DhcpServerv4Scope -Name "Guest-WiFi" `
    -StartRange 10.50.60.10 -EndRange 10.50.60.254 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 2.00:00:00  # 2-hour lease for guests

Set-DhcpServerv4OptionValue -ScopeId 10.50.60.0 `
    -DnsServer 8.8.8.8,8.8.4.4 `  # Google DNS for guests
    -Router 10.50.60.1

#========================================
# DHCP FAILOVER (ADDED MONTH 6)
#========================================
Add-DhcpServerv4Failover -Name "DHCP-Failover" `
    -PartnerServer "DC02.infogain.local" `
    -ScopeId 10.50.20.0,10.50.22.0,10.50.24.0,10.50.25.0 `
    -LoadBalancePercent 50 `
    -MaxClientLeadTime 01:00:00 `
    -AutoStateTransition $True `
    -StateSwitchInterval 01:00:00
```

**Building-by-Building Cutover Schedule:**

```
Week 1 (Friday Oct 22, 2021): Building A Cutover
├── 6:00 PM: User notification email sent
├── 6:30 PM: Updated old DHCP scope to include new DNS (10.50.10.10)
├── 7:00 PM: GPO script forced lease renewals (ipconfig /release && /renew)
├── 7:30 PM: Validated users pointing to new DNS
├── 8:00 PM: Activated new DHCP scope (10.50.20.0/23)
├── 8:30 PM: Deactivated old DHCP scope (172.16.10.0/24)
├── 9:00 PM: Monitored for issues
└── 10:00 PM: Cutover complete ✅ (400 users, zero issues)

Week 2 (Friday Oct 29, 2021): Building B Cutover
└── Result: 300 users migrated successfully ✅

Week 3 (Friday Nov 5, 2021): Buildings C & D Cutover
└── Result: 250 users migrated successfully ✅

FINAL RESULT: 850 users on new DNS/DHCP infrastructure
              Zero DNS/DHCP-related incidents post-migration
              Zero IP address conflicts (down from 47/month)
```

---

### Phase 6: Network Monitoring (Dec 2021 - Jan 2022)

#### PRTG Network Monitor Deployment

**Server Specifications:**
```
OS: Windows Server 2019
CPU: 4 vCPU
RAM: 16 GB
Disk: 250 GB SSD
IP: 10.50.0.50
Hostname: PRTG-MONITOR
Software: PRTG Network Monitor v21.4.73
License: 1000 sensors (576 initially configured)
```

**Sensor Distribution (576 Total):**

```
┌─────────────────────────────────────────────────────────┐
│ SWITCHES (42 devices × 8 sensors = 336 sensors)        │
├─────────────────────────────────────────────────────────┤
│ 1. Ping (availability)                                  │
│ 2. SNMP System Uptime                                   │
│ 3. SNMP Traffic (per interface)                         │
│ 4. SNMP CPU Load                                        │
│ 5. SNMP Memory                                          │
│ 6. SNMP Interface Status (up/down)                      │
│ 7. SNMP Port Status                                     │
│ 8. SNMP Custom (STP topology changes)                   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ CORE SWITCHES (2 devices × 15 sensors = 30 sensors)    │
├─────────────────────────────────────────────────────────┤
│ All standard switch sensors PLUS:                       │
│ 9. NetFlow (traffic analysis)                           │
│ 10. SNMP Stack Status                                   │
│ 11. SNMP VLAN Status                                    │
│ 12. SNMP OSPF Neighbors                                 │
│ 13. SNMP Routing Table                                  │
│ 14. HTTP (web interface)                                │
│ 15. Port Connectivity (each building link)              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ FIREWALL (1 device × 20 sensors = 20 sensors)          │
├─────────────────────────────────────────────────────────┤
│ 1. Ping                                                 │
│ 2. SNMP CPU Load                                        │
│ 3. SNMP Memory                                          │
│ 4. SNMP System Uptime                                   │
│ 5. SNMP Traffic WAN                                     │
│ 6. SNMP Traffic Internal                                │
│ 7. SNMP Concurrent Sessions                             │
│ 8. HTTP/HTTPS (web interface)                           │
│ 9. SNMP IPS Signature Updates                           │
│ 10-20. FortiGate-specific OIDs (VPN, licenses, etc.)   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ SERVERS (16 devices × 10 sensors = 160 sensors)        │
├─────────────────────────────────────────────────────────┤
│ 1. Ping                                                 │
│ 2. WMI CPU Load                                         │
│ 3. WMI Memory                                           │
│ 4. WMI Disk Free Space (per drive)                      │
│ 5. WMI System Uptime                                    │
│ 6. WMI Network Interface                                │
│ 7. WMI Services (critical services)                     │
│ 8. HTTP/HTTPS (web servers)                             │
│ 9. SQL Query (database servers)                         │
│ 10. SMTP (mail server)                                  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ DOMAIN CONTROLLERS (2 devices × 15 sensors = 30)       │
├─────────────────────────────────────────────────────────┤
│ All standard server sensors PLUS:                       │
│ 11. WMI Active Directory Replication                    │
│ 12. WMI DNS Service                                     │
│ 13. WMI DHCP Service                                    │
│ 14. WMI Netlogon Service                                │
│ 15. WMI LDAP Response Time                              │
└─────────────────────────────────────────────────────────┘
```

#### Alert Configuration

```
┌────────────────────────────────────────────────────────────┐
│ CRITICAL ALERTS (Email + SMS + ServiceNow Ticket)         │
├────────────────────────────────────────────────────────────┤
│ • Device down (ping fails 3 consecutive = 3 min)          │
│ • Core switch down or stack member failure                 │
│ • Firewall down                                            │
│ • Domain controller down                                   │
│ • Switch CPU > 90% for 5 minutes                           │
│ • Switch memory > 95%                                      │
│ • All interfaces down on a switch                          │
│ • AD replication failure                                   │
│                                                            │
│ Action:                                                    │
│   → Email: network-team@infogain.com                       │
│   → SMS: On-call engineer mobile                           │
│   → ServiceNow: Auto-create incident via API               │
│   → Average incident creation: 30 seconds                  │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ WARNING ALERTS (Email Only)                               │
├────────────────────────────────────────────────────────────┤
│ • Device CPU > 75% for 10 minutes                          │
│ • Memory > 85% for 10 minutes                              │
│ • Interface errors > 100 per hour                          │
│ • Interface utilization > 80% sustained                    │
│ • Disk space < 20% free                                    │
│ • DHCP scope utilization > 90%                             │
│                                                            │
│ Action:                                                    │
│   → Email: network-team@infogain.com                       │
│   → Investigation within 1 hour                            │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ INFORMATIONAL ALERTS (Daily Digest at 8 AM)              │
├────────────────────────────────────────────────────────────┤
│ • Average CPU > 50% over 24 hours                          │
│ • Interface flaps (up/down state changes)                  │
│ • Temperature sensors (if available)                       │
│ • Certificate expiration warnings (30 days)                │
│ • Firmware update available                                │
└────────────────────────────────────────────────────────────┘
```

#### Custom Dashboards

**Dashboard 1: Network Health Overview**
```
┌──────────────────────────────────────────────────┐
│ Campus Map (4 Buildings)                         │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐            │
│ │Bldg A│ │Bldg B│ │Bldg C│ │Bldg D│            │
│ │ 🟢   │ │ 🟢   │ │ 🟢   │ │ 🟢   │            │
│ │12 SW │ │10 SW │ │ 8 SW │ │ 6 SW │            │
│ └──────┘ └──────┘ └──────┘ └──────┘            │
├──────────────────────────────────────────────────┤
│ Top 10 Devices by CPU Utilization                │
│ 1. SWITCH-B-F3-IDF2: 45%                        │
│ 2. SWITCH-A-F2-IDF1: 38%                        │
│ 3. FGT-EDGE-01: 35%                             │
├──────────────────────────────────────────────────┤
│ Top 10 Interfaces by Bandwidth                   │
│ 1. Core-Gi1/0/1 (Bldg A): 145 Mbps             │
│ 2. Core-Gi1/0/2 (Bldg B): 98 Mbps              │
├──────────────────────────────────────────────────┤
│ Alerts (Last 24 Hours)                           │
│ Critical: 0  │ Warning: 3  │ Info: 12          │
└──────────────────────────────────────────────────┘
```

**Dashboard 2: Core Infrastructure**
```
┌──────────────────────────────────────────────────┐
│ Core Switch Stack Status                         │
│ Switch 1: Master  │ CPU: 15%  │ Mem: 48%        │
│ Switch 2: Member  │ CPU: 12%  │ Mem: 45%        │
│ Stack Status: Healthy ✓                          │
├──────────────────────────────────────────────────┤
│ FortiGate Firewall                               │
│ Status: Up (45 days)                             │
│ Throughput: IN: 85 Mbps │ OUT: 120 Mbps        │
│ Sessions: 12,543 / 500,000                       │
│ IPS: Enabled │ Signatures: Up to date           │
├──────────────────────────────────────────────────┤
│ Domain Controllers                                │
│ DC01: Up │ AD Replication: Healthy              │
│ DC02: Up │ Last Sync: 15 minutes ago            │
├──────────────────────────────────────────────────┤
│ DHCP Server (10.50.10.21)                       │
│ Bldg A: 245/380 (64% used)                      │
│ Bldg B: 198/380 (52% used)                      │
│ Bldg C: 102/155 (66% used)                      │
│ Bldg D: 67/155 (43% used)                       │
├──────────────────────────────────────────────────┤
│ DNS Server (10.50.10.10)                        │
│ Queries: 45,234 (last hour)                     │
│ Avg Response: 12ms                               │
│ Failed Queries: 23 (0.05%)                       │
└──────────────────────────────────────────────────┘
```

**Dashboard 3: Bandwidth Analysis**
```
┌──────────────────────────────────────────────────┐
│ Building-to-Building Traffic (NetFlow)          │
│                                                  │
│     Core Switch (10.50.0.1)                     │
│          │                                       │
│    ┌─────┼─────┬─────┬─────┐                   │
│    │     │     │     │     │                   │
│  Bldg A Bldg B Bldg C Bldg D Internet          │
│  145Mb  98Mb   67Mb   45Mb   180Mb             │
├──────────────────────────────────────────────────┤
│ Internet Bandwidth Utilization                   │
│ IN:  85 Mbps  │ Peak: 178 Mbps (89%)           │
│ OUT: 120 Mbps │ Peak: 195 Mbps (97%)           │
│                                                  │
│ Recommendation: Upgrade from 200 to 300 Mbps    │
├──────────────────────────────────────────────────┤
│ Top 10 Bandwidth Consumers (24h)                │
│ 1. 10.50.20.45: 12.4 GB (Video conference)     │
│ 2. 10.50.22.78: 8.9 GB (File transfers)        │
│ 3. 10.50.20.123: 7.2 GB (Cloud backup)         │
├──────────────────────────────────────────────────┤
│ Protocol Distribution                            │
│ HTTPS: 45%  │ HTTP: 20%  │ SMB: 15%           │
│ DNS: 5%     │ Other: 15%                        │
└──────────────────────────────────────────────────┘
```

#### ServiceNow Integration

```python
# PRTG to ServiceNow API Integration
# Configured in PRTG notification template

{
  "API_ENDPOINT": "https://infogaindev.service-now.com/api/now/table/incident",
  "METHOD": "POST",
  "HEADERS": {
    "Authorization": "Bearer <API_TOKEN>",
    "Content-Type": "application/json"
  },
  "BODY": {
    "short_description": "%device - %message",
    "description": "Device: %device\nSensor: %sensor\nMessage: %message\nValue: %lastvalue\nStatus: %laststatus\nTimestamp: %datetime",
    "urgency": "2",
    "impact": "2",
    "assignment_group": "Network Operations",
    "category": "Network",
    "u_source": "PRTG Monitoring"
  }
}

# Result: Critical alerts create incidents in 30 seconds
#         Reduced manual ticket creation by 90%
```

---

### Phase 7: Active Directory Migration (Feb-Apr 2022)

#### Migration Architecture

```
SOURCE DOMAINS (TO BE CONSOLIDATED):
┌────────────────────────────────────────────────┐
│ infogain-hq.local                              │
│ ├── 400 users                                  │
│ ├── 2x DCs (Windows Server 2008 R2)           │
│ └── Schema Level: 2008 R2                      │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│ infogain-dev.local                             │
│ ├── 300 users                                  │
│ ├── 1x DC (Windows Server 2012)               │
│ └── Schema Level: 2012                         │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│ infogain-support.local                         │
│ ├── 150 users                                  │
│ ├── 1x DC (Windows Server 2008)               │
│ └── Schema Level: 2008                         │
└────────────────────────────────────────────────┘

                    ↓ MIGRATION ↓

TARGET DOMAIN (NEW UNIFIED):
┌────────────────────────────────────────────────┐
│ infogain.local                                 │
│ ├── DC01: 10.50.10.10 (Building A)            │
│ │   └── FSMO Roles: Schema, Domain Naming,    │
│ │       RID, PDC Emulator                     │
│ ├── DC02: 10.50.10.11 (Building B - DR)       │
│ │   └── FSMO Role: Infrastructure Master      │
│ ├── Schema Level: Windows Server 2019         │
│ ├── Domain Functional Level: 2019             │
│ └── 850 users (after migration)               │
└────────────────────────────────────────────────┘
```

#### Migration Execution Timeline

```
PHASE 1: TRUST ESTABLISHMENT (Week 1)
┌────────────────────────────────────────────────┐
│ Created bidirectional forest trusts:          │
│ • infogain.local ↔ infogain-hq.local         │
│ • infogain.local ↔ infogain-dev.local        │
│ • infogain.local ↔ infogain-support.local    │
│                                                │
│ Validation:                                    │
│ • Test-ADTrust (PowerShell)                   │
│ • nltest /trusted_domains                     │
│ • Cross-domain authentication test            │
└────────────────────────────────────────────────┘

PHASE 2: PILOT MIGRATION - IT DEPT (Week 2-3)
┌────────────────────────────────────────────────┐
│ Saturday Feb 12, 2022: User Migration         │
│ ├── 8:00 AM: Started ADMT User Migration      │
│ ├── Tool: Active Directory Migration Tool     │
│ ├── Source: infogain-hq.local/IT-Staff OU    │
│ ├── Target: infogain.local/Users/IT OU       │
│ ├── Options:                                   │
│ │   ✓ Migrate SID history (file access)      │
│ │   ✓ Translate roaming profiles              │
│ │   ✓ Migrate passwords (PES)                 │
│ │   ✓ Update user rights                      │
│ │   ✓ Target state: Enabled                   │
│ ├── 9:45 AM: Migration completed              │
│ │   • Success: 48/50 users (96%)              │
│ │   • Failed: 2 users (duplicate UPN)         │
│ │   • Resolution: Fixed manually               │
│ └── 10:30 AM: Validation complete             │
│                                                │
│ Sunday Feb 13, 2022: Computer Migration       │
│ ├── 10:00 AM: Started ADMT Computer Migration │
│ ├── 50 IT workstations                        │
│ ├── Options:                                   │
│ │   ✓ Translate security on profiles          │
│ │   ✓ Restart after migration (5 min)         │
│ ├── 12:00 PM: All 50 computers migrated       │
│ │   • Success: 50/50 (100%)                   │
│ └── 12:30 PM: Users logged in successfully    │
│                                                │
│ Validation:                                    │
│ ✓ Profile loaded correctly                    │
│ ✓ File share access (SID history working)     │
│ ✓ Email (Outlook auto-reconfigured)           │
│ ✓ ServiceNow SSO working                      │
│ ✓ Printers still mapped                       │
└────────────────────────────────────────────────┘

PHASE 3: WAVE 2 - BUILDINGS A & D (Week 4-5)
┌────────────────────────────────────────────────┐
│ Saturday Feb 26, 2022: User Migration         │
│                                                │
│ Batch 1 (Bldg A, Floors 1-2): 100 users      │
│ ├── 8:00 AM - 9:30 AM                         │
│ ├── Success: 98/100 (98%)                     │
│ └── Failed: 2 (remigrated manually)            │
│                                                │
│ Batch 2 (Bldg A, Floors 3-4): 100 users      │
│ ├── 10:00 AM - 11:30 AM                       │
│ └── Success: 100/100 (100%)                    │
│                                                │
│ Batch 3 (Bldg A, Floors 5-6): 150 users      │
│ ├── 12:00 PM - 2:00 PM                        │
│ ├── Success: 147/150 (98%)                    │
│ └── Failed: 3 (password complexity - fixed)    │
│                                                │
│ Batch 4 (Bldg D, All floors): 100 users      │
│ ├── 2:30 PM - 4:00 PM                         │
│ └── Success: 98/100 (98%)                      │
│                                                │
│ Total Saturday: 443/450 users (98.4%)         │
│ ├── Remaining 7: Remigrated manually by 6 PM  │
│ └── Final: 450/450 (100%)                     │
│                                                │
│ Sunday Feb 27, 2022: Computer Migration       │
│ ├── Building D: 100 computers                 │
│ │   └── 10:00 AM - 11:30 AM (100% success)   │
│ ├── Building A: 350 computers                 │
│ │   └── 12:00 PM - 6:00 PM (99.4% success)   │
│ └── Outstanding: 2 laptops off-network        │
└────────────────────────────────────────────────┘

PHASE 4: WAVE 3 - BUILDINGS B & C (Week 7-8)
┌────────────────────────────────────────────────┐
│ Saturday Mar 12, 2022: User Migration         │
│ ├── Building B: 300 users                     │
│ │   └── Success: 297/300 (99%)               │
│ ├── Building C: 150 users                     │
│ │   └── Success: 147/150 (98%)               │
│ └── Total: 347/350 (99.1%)                    │
│                                                │
│ Sunday Mar 13, 2022: Computer Migration       │
│ ├── Building B: 300 computers                 │
│ │   └── Success: 295/300 (98.3%)             │
│ ├── Building C: 150 computers                 │
│ │   └── Success: 145/150 (96.7%)             │
│ └── Total: 345/350 (98.6%)                    │
│     Outstanding: 5 laptops off-network        │
└────────────────────────────────────────────────┘

PHASE 5: CLEANUP (Week 12)
┌────────────────────────────────────────────────┐
│ 30-day soak period completed                  │
│ ├── Monitored for issues                      │
│ ├── Help desk tickets: 47 (mostly education)  │
│ └── No critical migration issues              │
│                                                │
│ Decommissioning:                               │
│ ├── Removed forest trusts (Remove-ADTrust)    │
│ ├── Demoted old domain controllers            │
│ ├── Cleaned up DNS conditional forwarders     │
│ ├── Removed old DHCP scopes                   │
│ └── Archived old AD metadata                  │
└────────────────────────────────────────────────┘
```

#### Final Migration Statistics

```
┌─────────────────────────────────────────────────────────┐
│                   MIGRATION SUMMARY                      │
├─────────────┬───────┬───────────┬─────────┬─────────────┤
│   Domain    │ Users │ Computers │ Success │   Duration  │
├─────────────┼───────┼───────────┼─────────┼─────────────┤
│ infogain-hq │  400  │    395    │  99.2%  │ 2 weekends  │
├─────────────┼───────┼───────────┼─────────┼─────────────┤
│ infogain-dev│  300  │    297    │  99.0%  │ 1 weekend   │
├─────────────┼───────┼───────────┼─────────┼─────────────┤
│infogain-sup │  150  │    148    │  98.7%  │ 1 weekend   │
├─────────────┼───────┼───────────┼─────────┼─────────────┤
│   TOTAL     │  850  │    840    │  99.0%  │  10 days    │
└─────────────┴───────┴───────────┴─────────┴─────────────┘

Outstanding Items:
├── 8 users on extended leave (to migrate upon return)
├── 10 laptops off-network during migration weekends
└── All users successfully migrated by May 2022

Zero Data Loss ✓
Average Migration Time: 2 minutes per user
Help Desk Tickets: 47 (32 user education, 10 profile issues, 5 printer)
```

---

### Phase 8: Troubleshooting & Incident Response

#### Incident 1: Spanning Tree Loop (Building B Installation)

```
┌────────────────────────────────────────────────┐
│ INCIDENT: SPANNING TREE LOOP                   │
│ Date: November 20, 2021                        │
│ Duration: 15 minutes                           │
│ Impact: Building B - 300 users                 │
└────────────────────────────────────────────────┘

TIMELINE:
├── 13:45: Reported - "Network is down in Building B"
├── 13:46: Logged into core switch
│   └── Command: show spanning-tree
│       Result: Multiple topology changes, ports flapping
├── 13:48: Checked distribution switch
│   └── Command: show interfaces trunk
│       Result: Gi1/0/2 showing as "access" not "trunk"
├── 13:50: Root Cause Identified
│   └── Building B uplink configured as access port
│       Created Layer 2 loop (broadcast storm)
├── 13:52: Applied Fix
│   └── interface GigabitEthernet1/0/2
│       switchport mode trunk
│       switchport trunk allowed vlan 10,20,30,40,50
├── 13:54: Spanning tree reconverged
│   └── Loop eliminated, traffic restored
├── 13:57: Validated connectivity
│   └── Pinged 20 random workstations - all responded
└── 14:00: Incident closed

ROOT CAUSE:
Port Gi1/0/2 was in access mode instead of trunk mode
during installation. This caused traffic to loop between
distribution switch and core switch.

PREVENTION:
✓ Updated installation checklist
✓ Added mandatory trunk verification step:
  "show interfaces trunk" before connecting uplinks
✓ Created runbook for spanning tree troubleshooting

LESSONS LEARNED:
• Always verify trunk configuration before connecting
• Use "show spanning-tree summary" to detect loops
• Keep spare switch for quick replacement if needed
```

#### Incident 2: Rogue DHCP Server Detection (Month 5)

```
┌────────────────────────────────────────────────┐
│ INCIDENT: ROGUE DHCP SERVER DETECTED           │
│ Date: March 15, 2022                           │
│ Duration: 10 minutes investigation             │
│ Impact Prevented: Potential 150-user outage    │
└────────────────────────────────────────────────┘

TIMELINE:
├── 10:23: PRTG Alert - "Port Gi0/15 err-disabled"
├── 10:25: Logged into switch SWITCH-C-F2-IDF1
│   └── Command: show port-security interface Gi0/15
│       Result: DHCP snooping violation
├── 10:27: Checked DHCP snooping logs
│   └── Command: show ip dhcp snooping binding
│       Result: Unauthorized DHCP offers from MAC 00:11:22:33:44:55
├── 10:28: Physically located port
│   └── Building C, Floor 2, Port 15
│   └── Traced cable to cubicle C2-045
├── 10:30: Contacted user (Rajesh Kumar)
│   └── User: "I brought my home router for better WiFi"
│   └── Unplugged home router immediately
├── 10:32: Re-enabled port
│   └── interface GigabitEthernet0/15
│       shutdown
│       no shutdown
├── 10:33: Validated - port operational, no violations
└── 10:35: Incident closed, user educated

DETECTION METHOD:
DHCP snooping detected unauthorized DHCP server offering
IP addresses on the network. Port automatically shut down
(err-disabled) preventing network-wide IP conflicts.

IMPACT PREVENTED:
Without DHCP snooping, rogue DHCP server would have:
├── Issued incorrect IP addresses to 150+ users
├── Provided wrong default gateway (10.0.0.1 vs 10.50.24.1)
├── Caused complete loss of connectivity for Building C
└── Required 2-4 hours to identify and resolve manually

RESULT:
✓ DHCP snooping prevented major outage
✓ 10-minute investigation and resolution
✓ User educated on network policy
✓ Documented in KB article: "Rogue DHCP Prevention"

CONFIGURATION THAT SAVED US:
interface GigabitEthernet0/15
 ip dhcp snooping limit rate 10
 ip dhcp snooping vlan 32
```

#### Incident 3: Broadcast Storm (Month 7)

```
┌────────────────────────────────────────────────┐
│ INCIDENT: BROADCAST STORM                      │
│ Date: May 20, 2022                             │
│ Duration: 5 minutes                            │
│ Impact: Building C Floor 2 - 50 users          │
└────────────────────────────────────────────────┘

TIMELINE:
├── 14:12: PRTG Alert - "Interface Gi0/12 utilization 100%"
├── 14:13: Storm control automatically shut down port
│   └── SNMP trap sent to PRTG
│   └── Port status: err-disabled (storm-control)
├── 14:14: Logged into switch
│   └── Command: show interfaces Gi0/12
│       Result: 10,000+ broadcast packets/second
├── 14:15: Identified connected device
│   └── MAC address: AA:BB:CC:DD:EE:FF
│   └── IP address: 10.50.24.87 (DHCP binding table)
│   └── Hostname: WS-FINANCE-047
├── 14:16: Contacted user (Priya Sharma)
│   └── User: "My computer has been acting slow"
│   └── Scheduled for IT support
├── 14:17: Left port disabled, documented in ticket
└── 14:20: Workstation replaced next day

ROOT CAUSE:
Faulty network adapter (Intel I219-V) sending excessive
broadcast traffic. Hardware failure caused NIC to flood
network with ARP broadcasts.

IMPACT PREVENTED:
Storm control shut down port within 2 seconds, preventing:
├── Network congestion for entire Building C
├── Switch CPU spike (would have reached 95%+)
├── Potential spanning tree reconvergence
└── Service disruption for 150 users

CONFIGURATION THAT SAVED US:
interface range GigabitEthernet0/1-24
 storm-control broadcast level 10.00
 storm-control action shutdown
 storm-control action trap

RESULT:
✓ Storm contained to single port
✓ No impact to other 149 users in Building C
✓ Faulty workstation identified and replaced
✓ Port re-enabled after hardware replacement
```

---

## Measurable Results & ROI

### Performance Improvements

```
┌───────────────────────────────────────────────────────────┐
│                  BEFORE vs AFTER METRICS                   │
├─────────────────────────┬──────────┬──────────┬───────────┤
│       Metric            │  Before  │  After   │  Change   │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Network Uptime          │  97.2%   │  99.6%   │  +2.4%    │
│ Monthly Downtime        │ 24.5 hrs │  3.5 hrs │  -86%     │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ IP Address Conflicts    │ 47/month │ 0/month  │  -100%    │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ DNS Resolution Failures │ 20/week  │ 1-2/week │  -90%     │
│ Avg DNS Response Time   │  45 ms   │  12 ms   │  -73%     │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Help Desk Tickets       │180/month │ 58/month │  -68%     │
│ Network-related only    │          │          │           │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Switch Failures         │2-3/month │0.5/month │  -75%     │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Mean Time to Resolution │ 4.2 hrs  │  1.3 hrs │  -69%     │
│ (MTTR)                  │          │          │           │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Average Switch CPU      │ 25-45%   │ 12-18%   │  -38%     │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Building-to-Building    │ 15-25 ms │  2-5 ms  │  -80%     │
│ Latency                 │          │          │           │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ User Satisfaction       │ 2.8/5.0  │ 4.6/5.0  │  +64%     │
├─────────────────────────┼──────────┼──────────┼───────────┤
│ Security Incidents      │ 8/year   │ 0/6mo    │  -100%    │
│ (critical findings)     │          │          │           │
└─────────────────────────┴──────────┴──────────┴───────────┘
```

### Financial ROI Calculation

```
┌────────────────────────────────────────────────────────────┐
│                     ANNUAL COST SAVINGS                     │
├────────────────────────────────────────────────────────────┤
│                                                             │
│ 1. REDUCED DOWNTIME                                         │
│    Before: 24.5 hrs/month × ₹50,000/hour = ₹1,225,000     │
│    After:  3.5 hrs/month × ₹50,000/hour = ₹175,000         │
│    ─────────────────────────────────────────────────────   │
│    Monthly savings: ₹1,050,000                              │
│    Annual savings:  ₹12,600,000                             │
│                                                             │
│ 2. HELP DESK EFFICIENCY                                     │
│    Reduction: 122 tickets/month                            │
│    Time per ticket: 30 minutes                             │
│    Hours saved: 61 hours/month                             │
│    Cost: ₹500/hour                                         │
│    ─────────────────────────────────────────────────────   │
│    Monthly savings: ₹30,500                                 │
│    Annual savings:  ₹366,000                                │
│                                                             │
│ 3. ELIMINATED IP CONFLICT TROUBLESHOOTING                   │
│    Before: 47 conflicts/month                              │
│    Time per conflict: 2 hours                              │
│    Cost: ₹500/hour                                         │
│    ─────────────────────────────────────────────────────   │
│    Monthly savings: ₹47,000                                 │
│    Annual savings:  ₹564,000                                │
│                                                             │
│ 4. AVOIDED EMERGENCY REPAIRS                                │
│    Switch failures reduced: 2.5/month → 0.5/month          │
│    Emergency repair cost: ₹15,000 per incident             │
│    ─────────────────────────────────────────────────────   │
│    Monthly savings: ₹30,000                                 │
│    Annual savings:  ₹360,000                                │
│                                                             │
├────────────────────────────────────────────────────────────┤
│ TOTAL ANNUAL SAVINGS: ₹13,890,000 (~$167,000 USD)         │
├────────────────────────────────────────────────────────────┤
│                                                             │
│ PROJECT INVESTMENT                                          │
│ ├── Equipment: ₹12,200,000                                 │
│ ├── Labor & Installation: ₹300,000                         │
│ └── TOTAL COST: ₹12,500,000                                │
│                                                             │
│ PAYBACK PERIOD: 10.8 months                                │
│                                                             │
│ 3-YEAR ROI:                                                 │
│ ├── 3-year savings: ₹41,670,000                            │
│ ├── Investment: ₹12,500,000                                │
│ ├── Net benefit: ₹29,170,000                               │
│ └── ROI: 233%                                               │
└────────────────────────────────────────────────────────────┘
```

### Business Outcomes

```
OPERATIONAL IMPROVEMENTS:
✓ Single Active Directory domain (eliminated 3 identity silos)
✓ Centralized network management (1 monitoring tool vs. none)
✓ Standardized equipment (1 vendor platform vs. 4 mixed)
✓ Comprehensive documentation (27 KB articles, 6 major docs)
✓ Trained IT team (4 sessions, 20 participants)
✓ Automated configuration backups (daily vs. manual only)
✓ Proactive monitoring (576 sensors, 24/7 coverage)
✓ Predictable network performance (baselines established)

SECURITY IMPROVEMENTS:
✓ Port security on 640 access ports (prevented 23 rogue devices)
✓ DHCP snooping active (blocked 1 rogue DHCP server)
✓ Dynamic ARP Inspection deployed (dropped 12 spoofing attempts)
✓ Guest WiFi isolated (zero unauthorized internal access)
✓ SSH-only access (Telnet disabled on all 42 switches)
✓ SNMPv3 encryption (eliminated plaintext community strings)
✓ Firewall with IPS (blocked 2,347 attacks in 6 months)
✓ Configuration auditing (all changes logged and tracked)

COMPLIANCE & AUDIT:
✓ Passed ISO 27001 network controls audit (9.2/10 rating)
✓ Zero critical security findings
✓ 2 medium findings (resolved within 48 hours)
✓ Complete network documentation for compliance
✓ Change management process with approval workflow
```

---

## Technical Skills Demonstrated

### Networking
- Cisco IOS command-line configuration (Catalyst 2960X, 3850)
- VLAN design, creation, and inter-VLAN routing (Layer 2/3)
- OSPF dynamic routing protocol configuration
- Spanning Tree Protocol (RSTP) implementation and troubleshooting
- DNS/DHCP services architecture and deployment (Windows Server)
- IP addressing scheme design and subnet planning
- Network Address Translation (NAT/PAT) configuration
- ACL creation for traffic filtering and security
- Network performance optimization and capacity planning

### Network Security
- Port security with sticky MAC learning
- DHCP snooping (rogue DHCP server prevention)
- Dynamic ARP Inspection (anti-spoofing)
- Storm control (broadcast storm mitigation)
- FortiGate firewall policy configuration
- Intrusion Prevention System (IPS) deployment
- Guest network isolation with ACLs
- SSH hardening and access control
- SNMPv3 encryption implementation

### Monitoring & Troubleshooting
- PRTG Network Monitor deployment (576 sensors)
- SNMP/WMI/NetFlow sensor configuration
- Custom dashboard creation for multiple audiences
- Alert threshold tuning (reduced false positives by 80%)
- Syslog centralized logging and analysis
- ServiceNow ITSM integration via API
- Systematic troubleshooting using OSI model
- Root cause analysis and documentation
- Performance baseline establishment

### Systems Administration
- Active Directory domain services deployment (Windows Server 2019)
- ADMT (Active Directory Migration Tool) for domain consolidation
- 850 users migrated with 99% success rate
- PowerShell scripting for automation
- Group Policy Object (GPO) management
- DNS/DHCP Windows services administration

### Project Management
- Multi-phase project planning and execution
- Phased rollout strategy (minimized user disruption)
- Vendor coordination and procurement
- Risk identification and mitigation
- User communication and change management
- Team leadership (8-member team)
- Documentation and knowledge transfer

---

## Documentation Delivered

```
1. Network Architecture Diagram (Visio, 8 pages)
   ├── Physical topology with cable runs
   ├── Logical topology with VLANs and routing
   └── IP addressing scheme with all subnets

2. IP Address Management Spreadsheet (Excel)
   ├── All subnets with utilization tracking
   ├── DHCP scopes and ranges
   ├── Static IP assignments with purpose
   └── Updated weekly, version-controlled in Git

3. Switch Configuration Standards (Word, 25 pages)
   ├── Standard configuration templates
   ├── Naming conventions
   ├── Security settings
   ├── VLAN assignments by location
   └── Sample configurations for each switch type

4. Network Troubleshooting Guide (Confluence wiki)
   ├── Common issues and resolutions
   ├── Decision trees for systematic troubleshooting
   ├── Command cheat sheets
   └── Escalation procedures

5. Disaster Recovery Procedures (Word, 18 pages)
   ├── RTO/RPO targets (4 hours / 1 hour)
   ├── Hardware replacement procedures
   ├── Configuration restoration steps
   ├── Vendor contact information
   └── Emergency contact tree

6. Change Management Procedures (Confluence)
   ├── ServiceNow workflow documentation
   ├── Change approval matrix
   ├── Maintenance window guidelines
   └── Rollback procedures

7. Operational Runbooks (12 total)
   ├── Add new VLAN (30-minute procedure)
   ├── Replace failed switch (45-minute procedure)
   ├── Investigate network slowness
   ├── Configure DHCP reservation
   ├── Reset err-disabled port
   ├── Update switch firmware
   └── (6 additional runbooks)

8. Knowledge Base Articles (27 total)
   ├── KB001: Reset Switch Port (Err-Disabled)
   ├── KB002: Add DHCP Reservation for Printer
   ├── KB004: Troubleshoot "No IP Address" Issues
   ├── KB005: Interpret PRTG Alerts
   └── (23 additional articles)
```

---

## Key Accomplishments Summary

### Network Infrastructure
✅ Replaced 42 end-of-life switches with standardized Cisco equipment  
✅ Consolidated three separate networks (172.16/10.10/192.168) into unified 10.50.0.0/16  
✅ Designed and implemented 10-VLAN architecture for security segmentation  
✅ Deployed FortiGate firewall with IPS (blocked 2,347 attacks)  
✅ Configured Layer 3 routing with OSPF on core switch stack  

### DNS/DHCP
✅ Eliminated 47 monthly IP conflicts (100% reduction)  
✅ Centralized DNS/DHCP on Windows Server 2019 infrastructure  
✅ Improved DNS query response time by 73% (45ms → 12ms)  
✅ Deployed DHCP failover for high availability  

### Active Directory
✅ Migrated 850 users from 3 domains to 1 unified domain (99% success)  
✅ Migrated 840 computers with zero data loss  
✅ Average migration time: 2 minutes per user  
✅ Deployed new Windows Server 2019 DCs with replication  

### Security
✅ Implemented port security on 640 access ports (prevented 23 rogue devices)  
✅ DHCP snooping blocked rogue DHCP server (prevented major outage)  
✅ Dynamic ARP Inspection dropped 12 spoofing attempts  
✅ Guest WiFi isolation with ACLs (zero unauthorized access)  
✅ Passed ISO 27001 audit with 9.2/10 rating  

### Monitoring
✅ Deployed PRTG with 576 sensors covering 100% of infrastructure  
✅ Created 3 custom dashboards for different audiences  
✅ Reduced false positive alerts by 80% through baseline tuning  
✅ ServiceNow integration (30-second incident creation)  

### Business Impact
✅ Improved network uptime from 97.2% to 99.6%  
✅ Reduced help desk tickets by 68% (180 → 58 per month)  
✅ Reduced MTTR by 69% (4.2 hours → 1.3 hours)  
✅ User satisfaction increased 64% (2.8 → 4.6 out of 5.0)  
✅ ₹13.9M annual cost savings with 10.8-month payback period  

---

## Interview Talking Points

### "Tell me about a complex network problem you solved"

*"During Building B installation, we experienced a 15-minute outage affecting 300 users. I immediately logged into the core switch and ran 'show spanning-tree' which showed multiple topology changes—classic symptom of a Layer 2 loop. I then checked the distribution switch with 'show interfaces trunk' and discovered port Gi1/0/2 was in access mode instead of trunk mode. This created a broadcast loop between switches. I corrected it by configuring 'switchport mode trunk' and specifying allowed VLANs. Spanning tree reconverged within 2 minutes. To prevent recurrence, I updated our installation checklist to mandate trunk verification before connecting building uplinks. This systematic OSI model approach—identifying it as Layer 2, using appropriate commands, and implementing process improvements—is how I approach all network issues."*

### "Explain your VLAN design and network segmentation strategy"

*"I designed a 10-VLAN architecture based on security, performance, and management needs. Employee VLANs were segmented by building for easier troubleshooting—Building A got 10.50.20.0/23, Building B got 10.50.22.0/23, etc. Each VLAN has its own subnet with 20-30% growth headroom. Printers have VLAN 40 with DHCP reservations for consistent addressing. VoIP phones are in VLAN 50 with QoS priority configured to ensure call quality. The guest WiFi VLAN is completely isolated—I created an ACL that permits guest traffic to the internet but explicitly denies access to internal 10.50.0.0/16 networks. This prevented guests from accessing corporate resources. IoT devices like cameras are in VLAN 70 and blocked from internet access for security. On the core switch, I configured inter-VLAN routing using SVIs where each VLAN interface has a gateway IP. This design improved security through segmentation, performance through traffic isolation, and troubleshooting by having logical network divisions."*

### "Walk me through your network security implementation"

*"I implemented defense-in-depth with multiple security layers. At Layer 2, I configured port security on all 640 access ports limiting 2 MAC addresses per port—this prevents users from connecting unauthorized switches or hubs. I enabled DHCP snooping which actually saved us in Month 5 when a user accidentally connected a home router. The snooping detected unauthorized DHCP offers and automatically shut down the port, preventing a network-wide IP conflict storm that would have affected 150 users. I also enabled Dynamic ARP Inspection which validates ARP packets against the DHCP snooping binding table to prevent man-in-the-middle attacks—it dropped 12 spoofing attempts. At Layer 3, I used ACLs—management access to switches is restricted to the 10.50.0.0/24 and 10.50.10.0/24 subnets only, applied via 'access-class' on VTY lines. Guest WiFi is isolated via ACL that denies traffic to internal networks. On the FortiGate firewall, I implemented deny-all-by-default policies and explicitly allowed only required traffic. I also disabled insecure protocols: Telnet disabled, SSH v2 only, SNMPv3 with encryption. The firewall IPS blocked 2,347 attack attempts in 6 months. This layered approach means if one security control fails, others still provide protection."*

### "Describe your monitoring and alerting strategy"

*"I deployed PRTG Network Monitor with 576 sensors covering switches, routers, firewall, and servers. For switches, I monitor availability via ping, performance via SNMP CPU and memory, interface status and traffic, and security via port status checks. For the core switches, I added NetFlow sensors for bandwidth analysis showing top talkers and protocol distribution. I created three dashboards tailored to different audiences: a network health overview with a campus map color-coded by status for daily operations, a core infrastructure dashboard for critical systems, and a bandwidth analysis dashboard for capacity planning. I configured tiered alerting—critical alerts like device down or core switch failure trigger immediate SMS and automatically create ServiceNow incidents via API, reducing incident creation time to 30 seconds. Warning alerts like high CPU send emails for investigation within 1 hour. Informational alerts go into a daily digest. I established performance baselines during the first week by running in observation mode, then set alert thresholds at baseline plus 20% headroom. This reduced false positives by 80% and achieved a 5:1 actionable alert ratio. The monitoring proved invaluable—when DHCP snooping detected the rogue DHCP server, PRTG immediately alerted us, enabling 10-minute response time and preventing a major outage."*

---

**END OF PROJECT 1 REPORT**

---

# PROJECT 2: Azure Virtual Desktop (AVD) Enterprise Deployment

**Company:** Bank of New York  
**Role:** Systems Engineer - Infrastructure & Security Lead  
**Duration:** January 2024 - October 2025 (Present, 22 months)  
**Project Timeline:** January 2024 - June 2024 (6 months core implementation)  
**Budget:** $450,000  
**Team Size:** 5 members

---

## Executive Summary

Led enterprise-scale Azure Virtual Desktop deployment for 2,500+ employees with hybrid identity integration, zero-trust network security controls, and comprehensive monitoring. Replaced legacy Citrix infrastructure, reduced costs by 35% ($420K annual savings), improved network uptime from 97.2% to 99.94%, and achieved 11-second average login time (76% faster than legacy). Implemented Network Security Groups (NSGs) as firewall policies, configured hybrid VPN connectivity, and responded to security incident within 30 minutes preventing data breach.

---

## Business Challenge

Bank of New York required secure remote access solution for hybrid workforce while maintaining financial industry compliance (SOC 2, PCI DSS). Legacy Citrix infrastructure was costly ($1.2M/year), unreliable (97.2% uptime, 45-second login times), and lacked modern security features like conditional access and MFA.

**Problems with Legacy System:**
- High annual cost: $1.2M for Citrix licenses and hardware
- Poor user experience: 45-second average login time
- Limited security: No MFA, no conditional access, no geo-blocking
- Frequent outages: 97.2% uptime (24+ hours monthly downtime)
- Help desk burden: 80 network/VDI tickets per month
- No BYOD support: Employees couldn't work from personal devices

---

## Your Key Responsibilities & Technical Achievements

### Phase 1: Network Architecture Design (Weeks 1-3)

#### Azure Virtual Network Design

```
AZURE VNETS (EAST US 2 PRIMARY REGION)
┌────────────────────────────────────────────────────────┐
│ Hub VNet: 10.100.0.0/24                                │
│ ├── Gateway Subnet: 10.100.0.0/27                     │
│ │   └── VPN Gateway + ExpressRoute Gateway            │
│ └── Azure Bastion Subnet: 10.100.0.64/26              │
├────────────────────────────────────────────────────────┤
│ AVD VNet: 10.100.10.0/23 (Session Hosts)              │
│ ├── Subnet-AVD-SessionHosts: 10.100.10.0/24           │
│ │   └── 290 session host VMs (Room for 254)           │
│ └── Subnet-AVD-HostPool2: 10.100.11.0/24              │
│     └── Reserved for growth                            │
├────────────────────────────────────────────────────────┤
│ Management VNet: 10.100.20.0/24                        │
│ ├── Domain Controllers: 10.100.20.10, 10.100.20.11    │
│ ├── Management Tools: 10.100.20.50                    │
│ └── Jump Hosts: 10.100.20.100-110                     │
├────────────────────────────────────────────────────────┤
│ Storage VNet: 10.100.30.0/24                           │
│ └── NetApp Azure Files: 10.100.30.10                  │
│     └── FSLogix Profile Containers (50 TB)            │
└────────────────────────────────────────────────────────┘

         ↕ VNet Peering (Hub-Spoke Model)

ON-PREMISES DATA CENTER (HYBRID CONNECTIVITY)
┌────────────────────────────────────────────────────────┐
│ Corporate Network: 192.168.0.0/16                      │
│ ├── Domain Controllers: 192.168.10.10, 192.168.10.11  │
│ ├── File Servers: 192.168.20.0/24                     │
│ └── User Workstations: 192.168.100.0/22               │
└────────────────────────────────────────────────────────┘

CONNECTIVITY:
├── Primary: Site-to-Site VPN (IPsec)
│   └── Bandwidth: 1 Gbps, Latency: 8ms average
└── Secondary: ExpressRoute (Automatic Failover)
    └── Bandwidth: 1 Gbps, 99.95% SLA
```

**Design Rationale:**
- Hub-spoke VNet architecture for centralized management
- Separate VNets for security isolation (AVD, Management, Storage)
- /23 for AVD allows 510 IPs (current: 290 VMs, 43% headroom)
- Hybrid connectivity for Active Directory integration
- Azure Bastion for secure RDP access (no public IPs on VMs)

---

### Phase 2: Network Security Implementation (Weeks 4-9)

#### Network Security Groups (NSG) Configuration

**NSG: AVD-SessionHosts-NSG**

```
PRIORITY │ NAME                    │ SOURCE          │ DEST    │ PORT    │ ACTION
─────────┼─────────────────────────┼─────────────────┼─────────┼─────────┼────────
100      │ Allow-AzureBastion-RDP  │ AzureBastionSub │ *       │ 3389    │ Allow
200      │ Allow-AVD-Service       │ *               │ WVDSvc  │ 443     │ Allow
300      │ Allow-DC-Kerberos       │ *               │ 192.168.│ 88      │ Allow
         │                         │                 │ 10.10   │         │
310      │ Allow-DC-LDAP          │ *               │ 192.168.│ 389,636 │ Allow
         │                         │                 │ 10.10   │         │
320      │ Allow-DC-DNS           │ *               │ 192.168.│ 53      │ Allow
         │                         │                 │ 10.10   │         │
400      │ Allow-NetApp-SMB       │ *               │ 10.100. │ 445     │ Allow
         │                         │                 │ 30.10   │         │
500      │ Allow-Azure-Monitor    │ *               │ AzureMon│ 443     │ Allow
4000     │ Deny-Internet-Outbound │ *               │ Internet│ *       │ Deny
4096     │ Deny-All-Inbound       │ *               │ *       │ *       │ Deny
```

**Key Security Principles:**
- **Deny-all-by-default:** Last rule explicitly denies everything
- **Least privilege:** Only allow required ports and protocols
- **No direct internet:** Session hosts cannot browse internet directly
- **Service tags:** Use Azure service tags (WindowsVirtualDesktop, AzureMonitor)
- **Stateful firewall:** NSGs are stateful (return traffic automatically allowed)

**NSG Flow Logs Integration:**
```powershell
# Enabled NSG flow logs for security monitoring
$nsg = Get-AzNetworkSecurityGroup -Name "AVD-SessionHosts-NSG"
Set-AzNetworkWatcherFlowLogConfig `
    -NetworkWatcher $networkWatcher `
    -TargetResourceId $nsg.Id `
    -StorageAccountId $storageAccount.Id `
    -EnableFlowLog $true `
    -FormatVersion 2 `
    -EnableTrafficAnalytics $true `
    -TrafficAnalyticsWorkspaceId $lawWorkspace.ResourceId
```

**Result:** Zero unauthorized inbound connections in 22 months

---

#### VPN & Hybrid Connectivity Configuration

**Azure VPN Gateway Setup:**

```powershell
#=========================================
# SITE-TO-SITE VPN CONFIGURATION
#=========================================
# VPN Gateway: VpnGw2 SKU (1.25 Gbps throughput)

# Create VPN Gateway
$gwSubnet = Get-AzVirtualNetworkSubnetConfig `
    -Name "GatewaySubnet" `
    -VirtualNetwork $hubVnet

$gwIp = New-AzPublicIpAddress `
    -Name "VNG-Hub-EastUS2-IP" `
    -ResourceGroupName $rgName `
    -Location "EastUS2" `
    -AllocationMethod "Static" `
    -Sku "Standard"

$gwIpConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "VNG-IPConfig" `
    -SubnetId $gwSubnet.Id `
    -PublicIpAddressId $gwIp.Id

New-AzVirtualNetworkGateway `
    -Name "VNG-Hub-EastUS2" `
    -ResourceGroupName $rgName `
    -Location "EastUS2" `
    -IpConfigurations $gwIpConfig `
    -GatewayType "Vpn" `
    -VpnType "RouteBased" `
    -GatewaySku "VpnGw2" `
    -EnableActiveActiveFeature $true

# Create Local Network Gateway (On-premises)
New-AzLocalNetworkGateway `
    -Name "LNG-OnPrem-PrimaryDC" `
    -ResourceGroupName $rgName `
    -Location "EastUS2" `
    -GatewayIpAddress "203.0.113.50" `
    -AddressPrefix @("192.168.0.0/16")

# Create VPN Connection
$vpnConnection = New-AzVirtualNetworkGatewayConnection `
    -Name "VPN-Azure-to-OnPrem" `
    -ResourceGroupName $rgName `
    -Location "EastUS2" `
    -VirtualNetworkGateway1 $vpnGateway `
    -LocalNetworkGateway2 $localGateway `
    -ConnectionType "IPsec" `
    -SharedKey "ComplexPr3Sh@redK3y!2024" `
    -EnableBgp $false

# IPsec parameters for security
Set-AzVirtualNetworkGatewayConnection `
    -VirtualNetworkGatewayConnection $vpnConnection `
    -IpsecPolicies @(
        New-AzIpsecPolicy `
            -IkeEncryption "AES256" `
            -IkeIntegrity "SHA384" `
            -DhGroup "DHGroup24" `
            -IpsecEncryption "GCMAES256" `
            -IpsecIntegrity "GCMAES256" `
            -PfsGroup "PFS24" `
            -SALifeTimeSeconds 3600 `
            -SADataSizeKilobytes 102400000
    )
```

**VPN Performance Metrics:**
```
Bandwidth: 1.0 Gbps (VpnGw2 SKU)
Latency: 8ms average (Azure East US 2 to on-premises)
Jitter: <2ms
Packet Loss: <0.01%
Concurrent Connections: 290 session hosts + 200 VPN users
Uptime: 99.98% (tested with failover to ExpressRoute)
```

**Failover Test Results:**
```
Scenario: Primary VPN gateway failure simulation
├── T+0:00: Manually disabled primary VPN gateway
├── T+0:45: BGP detected route unavailability
├── T+0:50: Traffic automatically rerouted to ExpressRoute
├── T+1:00: All session hosts reconnected via ExpressRoute
└── Result: 45-second failover, zero session disconnections
         (Users experienced brief reconnect, no data loss)
```

---

#### DNS/DHCP Hybrid Configuration

**DNS Architecture:**

```
AZURE DNS (CONDITIONAL FORWARDING)
┌────────────────────────────────────────┐
│ Azure VNet DNS: 168.63.129.16 (Azure)  │
│ Custom DNS Servers:                     │
│ ├── Primary: 192.168.10.10 (On-prem)  │
│ └── Secondary: 192.168.10.11 (On-prem)│
└────────────────────────────────────────┘

         ↓ DNS Queries Flow ↓

ON-PREMISES DNS SERVERS
┌────────────────────────────────────────┐
│ Primary DNS: DC01 (192.168.10.10)      │
│ Secondary DNS: DC02 (192.168.10.11)    │
│                                        │
│ Zones:                                 │
│ ├── Forward: bnymellon.corp           │
│ │   └── DC01, DC02, File servers      │
│ ├── Reverse: 192.168.in-addr.arpa    │
│ └── Conditional Forwarder:            │
│     └── Azure Private DNS              │
└────────────────────────────────────────┘
```

**Azure VNet DNS Configuration:**

```powershell
# Configure custom DNS servers for AVD VNet
$avdVnet = Get-AzVirtualNetwork -Name "VNet-AVD-EastUS2"
$avdVnet.DhcpOptions.DnsServers = @("192.168.10.10", "192.168.10.11")
Set-AzVirtualNetwork -VirtualNetwork $avdVnet

# Validate DNS resolution from AVD subnet
Test-NetConnection -ComputerName "dc01.bnymellon.corp" -Port 389
# Result: Success, 8ms latency

nslookup dc01.bnymellon.corp 192.168.10.10
# Result: 192.168.10.10, 15ms response time
```

**DHCP Configuration:**
```
Azure VMs use Azure-provided DHCP (automatic)
├── IP assignment: Dynamic from subnet range
├── DNS servers: Configured at VNet level (192.168.10.10, .11)
├── Domain suffix: bnymellon.corp
└── Lease time: Infinite (VM lifetime)

Session Host IP Example:
├── VM Name: AVD-HP-Finance-045
├── IP Address: 10.100.10.45/24 (Azure DHCP assigned)
├── Gateway: 10.100.10.1 (Azure)
├── DNS Servers: 192.168.10.10, 192.168.10.11 (On-prem)
└── Domain: bnymellon.corp (Hybrid Azure AD joined)
```

---

### Phase 3: Monitoring & Incident Response (Weeks 15-17, Ongoing)

#### Azure Monitor & Log Analytics Configuration

**KQL Alert Queries Created (15 Total):**

**Alert 1: Failed Authentication Detection**
```kusto
// Detects brute force or credential stuffing attacks
SigninLogs
| where TimeGenerated > ago(1h)
| where AppDisplayName == "Windows Virtual Desktop"
| where ResultType != 0  // Failed logins
| summarize FailedLogins = count() by UserPrincipalName, IPAddress, Location
| where FailedLogins > 10
| project TimeGenerated, UserPrincipalName, IPAddress, Location, FailedLogins
```

**Alert 2: Unusual Geographic Login**
```kusto
// Detects logins from blocked or unusual countries
SigninLogs
| where TimeGenerated > ago(5m)
| where AppDisplayName == "Windows Virtual Desktop"
| where ResultType == 0  // Successful login
| where Location in ("Russia", "China", "North Korea", "Iran")
| project TimeGenerated, UserPrincipalName, IPAddress, Location, DeviceDetail
```

**Alert 3: Session Host Performance Degradation**
```kusto
// Detects session hosts with high CPU impacting user experience
Perf
| where TimeGenerated > ago(15m)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue) by Computer
| where AvgCPU > 85
| project Computer, AvgCPU, Threshold = 85
```

**Alert 4: FSLogix Profile Load Delay**
```kusto
// Detects slow profile loading affecting login times
WVDCheckpoints
| where TimeGenerated > ago(10m)
| where Name contains "FSLogix"
| where Parameters contains "ProfileLoadTime"
| extend LoadTime = extract("LoadTime:(\\d+)", 1, Parameters)
| where toint(LoadTime) > 30000  // 30 seconds
| project TimeGenerated, Computer, UserName, LoadTime
```

**Alert 5: Network Latency Issues**
```kusto
// Detects high latency between Azure and on-premises
AzureMetrics
| where TimeGenerated > ago(10m)
| where ResourceProvider == "MICROSOFT.NETWORK"
| where MetricName == "AverageRoundtripMs"
| summarize AvgLatency = avg(Average) by Resource
| where AvgLatency > 150  // 150ms threshold
| project Resource, AvgLatency, Threshold = 150
```

**Alert Configuration Matrix:**

| Alert Name | Query | Threshold | Severity | Action | Frequency |
|------------|-------|-----------|----------|--------|-----------|
| Failed Authentications | SigninLogs | >10/hour | High | Email + SMS | Every 5 min |
| Geo-Blocked Login | SigninLogs | Any | Critical | Email + SMS + ServiceNow | Real-time |
| High CPU | Perf | >85% for 10min | Medium | Email | Every 15 min |
| Profile Load Delay | WVDCheckpoints | >30 seconds | Medium | Email | Every 10 min |
| Network Latency | AzureMetrics | >150ms | Medium | Email | Every 10 min |
| Session Host Down | Heartbeat | No heartbeat 5min | Critical | Email + SMS | Every 5 min |

---

#### Security Incident Response Example (Month 3)

```
┌────────────────────────────────────────────────────────┐
│ INCIDENT: SUSPECTED ACCOUNT COMPROMISE                 │
│ Date: March 18, 2024, 2:15 AM                          │
│ Duration: 30 minutes (detection to containment)        │
│ Impact: Zero data breach, zero unauthorized access     │
└────────────────────────────────────────────────────────┘

TIMELINE:
├── 02:15 AM: Azure Monitor Alert Triggered
│   └── "User jsmith@bnymellon.com login attempt from Russia"
│   └── Source IP: 185.220.101.45 (Tor exit node)
│   └── User Agent: Chrome 120.0 / Windows 10
│
├── 02:15 AM: Conditional Access Policy AUTO-BLOCKED
│   └── Geo-blocking rule denied access automatically
│   └── User never gained access to AVD environment
│
├── 02:17 AM: On-Call Engineer (You) Paged via SMS
│   └── "CRITICAL: Suspicious login attempt - Russia"
│
├── 02:20 AM: Investigation Began
│   └── Logged into Azure Portal → Azure AD Sign-in Logs
│   └── Query: UserPrincipalName == "jsmith@bnymellon.com"
│
├── 02:25 AM: Pattern Identified
│   └── Found 15 failed password attempts in last hour
│   └── Multiple user accounts targeted (password spray attack)
│   └── Source IPs: Russia, China, Ukraine (Tor network)
│   └── Targeted users: jsmith, rjohnson, mbrown, skumar, +11 others
│
├── 02:30 AM: SIEM Correlation Check
│   └── Checked for lateral movement: None found
│   └── Checked for data exfiltration: No unusual data transfers
│   └── Verified: No successful logins from suspicious IPs
│
├── 02:35 AM: Immediate Actions Taken
│   ├── Reset jsmith password (forced at next login)
│   ├── Revoked all active sessions for jsmith
│   │   └── Revoke-AzureADUserAllRefreshToken -ObjectId <user-id>
│   ├── Added source IPs to Azure Firewall block list
│   │   └── 15 IPs added to "Blocked-Threat-IPs" IP Group
│   └── Disabled jsmith account temporarily (until user contacted)
│
├── 02:45 AM: Documentation & Notification
│   ├── Created incident ticket: INC-2024-00315 (ServiceNow)
│   ├── Notified Security Operations Center (SOC)
│   ├── Documented timeline with screenshots
│   └── Prepared communication for user
│
├── 08:00 AM: User Contact & Recovery
│   ├── Called jsmith (confirmed legitimate user)
│   ├── User confirmed: Clicked phishing email link 2 days ago
│   ├── Required: Re-enroll MFA with new device
│   ├── Security awareness training scheduled
│   └── Re-enabled account with new password
│
├── 09:00 AM: Enhanced Monitoring
│   ├── Added alert: >3 failed attempts from single IP → block
│   ├── Created alert: Multiple users targeted from same IP
│   └── Increased monitoring for jsmith account (30 days)
│
└── 10:00 AM: Incident Closed
    ├── No data breach confirmed
    ├── No unauthorized access occurred
    ├── User account secured
    └── Post-incident report completed

ROOT CAUSE:
User clicked phishing link in email received March 16
└── Phishing site harvested credentials
    └── Attackers attempted access 48 hours later
        └── Geo-blocking prevented access

WHAT PREVENTED BREACH:
✓ Conditional Access geo-blocking (Russia auto-denied)
✓ Azure Monitor real-time alerting (2-minute detection)
✓ MFA requirement (even with password, MFA challenge failed)
✓ NSG restrictions (even if authenticated, limited access)

LESSONS LEARNED:
1. Geo-blocking is effective (prevented unauthorized access)
2. Real-time monitoring critical (30-minute containment)
3. User education needed (phishing awareness)
4. MFA is essential (password alone insufficient)

IMPROVEMENTS IMPLEMENTED:
✓ Enhanced email filtering rules (block similar phishing)
✓ Added alert: Failed auth across multiple accounts
✓ Mandatory security awareness training (monthly)
✓ FIDO2 security keys for all administrators
✓ Reduced Conditional Access session timeout (8hrs → 4hrs)
```

**Alert Configuration After Incident:**

```kusto
// NEW ALERT: Distributed Password Spray Detection
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType != 0  // Failed logins
| summarize 
    TargetedUsers = dcount(UserPrincipalName),
    FailedAttempts = count()
    by IPAddress
| where TargetedUsers > 5  // Same IP targeting 5+ users
| project IPAddress, TargetedUsers, FailedAttempts
```

---

## Measurable Results & ROI

```
┌───────────────────────────────────────────────────────────┐
│               BEFORE (CITRIX) vs AFTER (AVD)               │
├────────────────────────┬──────────┬──────────┬────────────┤
│        Metric          │  Before  │  After   │   Change   │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Annual Infra Cost      │ $1.2M    │ $780K    │ -35% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Network Uptime         │ 97.2%    │ 99.94%   │ +2.7% ✓    │
│ Monthly Downtime       │ 24.5 hrs │ 0.4 hrs  │ -98% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Average Login Time     │ 45 sec   │ 11 sec   │ -76% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ User Satisfaction      │ 3.2/5.0  │ 4.6/5.0  │ +44% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Help Desk Tickets      │ 80/month │ 32/month │ -60% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Security Score         │ 62%      │ 91%      │ +29pts ✓   │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Security Incidents     │ 3/year   │ 0 (22mo) │ -100% ✓    │
│ (Critical)             │          │          │            │
├────────────────────────┼──────────┼──────────┼────────────┤
│ MFA Adoption           │ 0%       │ 98%      │ +98pts ✓   │
├────────────────────────┼──────────┼──────────┼────────────┤
│ Supported Users        │ 2,000    │ 2,500    │ +25% ✓     │
├────────────────────────┼──────────┼──────────┼────────────┤
│ BYOD Support           │ No       │ 450 users│ NEW ✓      │
└────────────────────────┴──────────┴──────────┴────────────┘

FINANCIAL ROI:
├── Annual Savings: $420,000
├── 3-Year Savings: $1,260,000
├── Project Investment: $450,000
├── Payback Period: 13 months
└── 3-Year ROI: 180%

SECURITY IMPROVEMENTS:
├── Blocked Login Attempts: 1,247 (geo-blocking + conditional access)
├── MFA Adoption: 98% (2,450 users)
├── Password Spray Attacks Blocked: 3
├── Credential Compromise Attempts: 1 (contained in 30 minutes)
├── Penetration Test Results: 9.2/10 (zero critical findings)
└── Compliance Audits Passed: SOC 2, PCI DSS (zero findings)
```

---

## Technical Skills Demonstrated

### Networking
- Azure Virtual Network design and IP address planning
- Network Security Groups (NSGs) configuration as stateful firewalls
- Site-to-Site VPN configuration with IPsec
- ExpressRoute integration for hybrid connectivity
- DNS/DHCP configuration in hybrid environments
- Network latency optimization (Azure Accelerated Networking)
- VNet peering (hub-spoke architecture)
- NSG flow logs integration with Log Analytics

### Security
- Zero-trust network architecture implementation
- Conditional Access policies with MFA enforcement
- Network Security Groups (firewall rules) configuration
- Geo-blocking and IP restriction policies
- Security incident detection and response (30-minute containment)
- Vulnerability assessment and remediation (Qualys)
- Azure Firewall policy configuration
- Compliance validation (SOC 2, PCI DSS)

### Monitoring & Troubleshooting
- Azure Monitor and Log Analytics workspace configuration
- KQL (Kusto Query Language) query writing for security events
- Alert creation, threshold tuning, and escalation workflows
- Dashboard design for executives, operations, and security teams
- Performance baseline establishment
- Network latency troubleshooting (VPN, ExpressRoute)
- DNS resolution troubleshooting in hybrid scenarios

### Cloud Infrastructure
- Azure Virtual Desktop (AVD) architecture and deployment
- Azure AD Connect configuration (hybrid identity)
- FSLogix profile management with NetApp Azure Files
- Virtual machine deployment and configuration at scale (290 VMs)
- Autoscaling configuration for cost optimization
- Load testing and capacity planning

---

## Interview Talking Points

### "Describe your network troubleshooting experience in Azure"

*"When users in the Finance department couldn't access AVD, I used a systematic OSI model approach. First, I verified Layer 3 connectivity by checking NSG flow logs—traffic was reaching the Azure VPN Gateway successfully. Then I tested DNS resolution from an AVD session host using nslookup. Users could ping 192.168.10.10 by IP but not dc01.bnymellon.corp by hostname. This indicated a DNS issue. I checked the Azure VNet DNS configuration and found it was still pointing to Azure-provided DNS (168.63.129.16) instead of our on-premises DNS servers. I updated the VNet DNS settings to use 192.168.10.10 and .11, forced DHCP renewals on the session hosts, and DNS resolution was restored within 15 minutes. I then updated our deployment documentation to include DNS validation as a mandatory checkpoint."*

### "Explain your Azure NSG (firewall) configuration strategy"

*"I configured Network Security Groups which function as stateful firewalls in Azure. For AVD session hosts, I implemented a zero-trust, deny-all-by-default approach. Inbound, I only allowed RDP from Azure Bastion (no direct internet RDP exposure). Outbound, I explicitly allowed only required traffic: AVD service endpoints on TCP 443 using the WindowsVirtualDesktop service tag, on-premises domain controller traffic for Kerberos (88), LDAP (389, 636), and DNS (53), and NetApp file storage for FSLogix profiles on SMB port 445. I explicitly denied all other internet egress. I enabled NSG flow logs and integrated them with Log Analytics for security monitoring. When reviewing flow logs, I identified users trying to access blocked websites, which validated our security posture. This granular control resulted in zero unauthorized network access in 22 months of operation."*

### "Walk me through your security incident response process"

*"When Azure Monitor alerted me at 2:15 AM about a login attempt from Russia for user jsmith, I immediately investigated using Azure AD Sign-in Logs. The Conditional Access geo-blocking policy had already automatically denied the access, but I still needed to determine if this was a compromised account. I queried for jsmith's recent activity and found 15 failed password attempts in the previous hour, plus 14 other users targeted from the same IP range—classic signs of a password spray attack. I checked for any successful logins from suspicious IPs (none found), verified no data exfiltration or lateral movement occurred by reviewing Azure Storage logs and session host activity. Within 30 minutes, I reset jsmith's password, revoked all active sessions using PowerShell (Revoke-AzureADUserAllRefreshToken), added the attacker IPs to our Azure Firewall block list, and created an incident ticket with full timeline documentation. At 8 AM, I contacted the user, confirmed they clicked a phishing link 2 days prior, required MFA re-enrollment, and scheduled security awareness training. The result: zero data breach, zero unauthorized access, because our layered security (geo-blocking + MFA + monitoring) worked as designed. I then enhanced our detection by creating an alert that triggers when a single IP targets multiple user accounts."*

### "How do you approach network monitoring in Azure?"

*"I built comprehensive monitoring using Azure Monitor and Log Analytics. I created 15 KQL queries to detect security and performance issues: failed authentications above threshold, logins from blocked countries, session host CPU exceeding 85%, slow FSLogix profile loads, and network latency issues. I configured tiered alerting—critical alerts like geo-blocked logins or session host failures trigger immediate SMS notifications, while warning alerts like high CPU send emails for investigation within an hour. I created three dashboards tailored to different audiences: an executive dashboard showing active users, costs, uptime, and security incidents; an operations dashboard with session host health, resource utilization, and network latency; and a security dashboard tracking failed authentications, conditional access blocks, and MFA success rates. I established performance baselines during the first month—average login time 11 seconds, CPU 68%, latency 9ms—then set alert thresholds at baseline plus 20% to minimize false positives. This proactive monitoring enabled 30-minute response to the Russian login attempt and has maintained 99.94% uptime for 22 months."*

---

**END OF PROJECT 2 REPORT**
```

This markdown format is optimized for PDF conversion and printing, with clear section breaks, code blocks, tables, and hierarchical structure. Both projects are complete with all technical details, metrics, and interview talking points.
