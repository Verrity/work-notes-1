
---
### Теги

### [[Конфигурации WLC|Назад]]
### Далее
####
---


```cfg title="WLC"
#!/usr/bin/clish
#362
#1.36.4
#2026-05-15
#11:43:51
cluster
  cluster-interface bridge 2
  unit 1
    mac-address 68:13:e2:7e:82:46
  exit
  unit 2
    mac-address 90:54:b7:3b:a1:40
  exit
  enable
exit

hostname wlc-1 unit 1
hostname wlc-2 unit 2

object-group network SYNC_DEST
  ip address-range 192.168.2.12 unit 1
  ip address-range 192.168.2.11 unit 2
exit
object-group network SYNC_SRC
  ip address-range 192.168.2.11 unit 1
  ip address-range 192.168.2.12 unit 2
exit

syslog max-files 3
syslog file-size 512
syslog file tmpsys:syslog/default
  severity debug
exit
syslog file tmpsys:syslog/radius_log
  severity debug
  match process-name radius-server
exit
syslog console
  severity warning
exit

logging radius

radius-server local
  nas ap-shared
    key ascii-text encrypted 8CB5107EA7005AFF
    network 192.168.5.0/24
  exit
  nas ap1
    key ascii-text encrypted 8CB5107EA7005AFF
    network 192.168.3.0/24
  exit
  nas ap2
    key ascii-text encrypted 8CB5107EA7005AFF
    network 192.168.4.0/24
  exit
  nas local
    key ascii-text encrypted 8CB5107EA7005AFF
    network 127.0.0.1/32
  exit
  domain default
  exit
  domain wlc.root
    user tester
      password ascii-text encrypted 8CB5107EA7005AFF
    exit
  exit
  virtual-server default
    enable
  exit
  enable
exit
username admin
  password encrypted $6$bi0rHDrfrs6pIM0E$TbYyX/n228dlUmA3Ky2k1dKpmft.FAz9ql3Mpy2gO7.aNK8PVN5iAe3iQZfCh5yINNztSflGEFzrlze5wr4in/
exit
username techsupport
  password encrypted $6$NlB/GvAp4WiYzd/r$4Wi326ZU2C4U1vxJdszIsdXs0QXudD.ICnkHAp33DLyP/KVRWj/x7lbFybNU6qtRhtAYe9/mKvR5SmHM52.Z91
  mode techsupport
exit

radius-server host 127.0.0.1
  key ascii-text encrypted 8CB5107EA7005AFF
exit
aaa radius-profile default_radius
  radius-server host 127.0.0.1
exit

vlan 100
  force-up
exit
vlan 200
  force-up
exit
vlan 300
  force-up
exit
vlan 400
  force-up
exit
vlan 500
  force-up
exit
vlan 600
  force-up
exit

no spanning-tree

domain lookup enable

security zone trusted
exit
security zone untrusted
exit
security zone users
exit

bridge 1
  description "Untagged"
  vlan 100
  ip firewall disable
  ip address 192.168.1.11/24 unit 1
  ip address 192.168.1.12/24 unit 2
  vrrp 1
    ip address 192.168.1.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit
bridge 2
  description "SYNC"
  vlan 200
  ip firewall disable
  ip address 192.168.2.11/24 unit 1
  ip address 192.168.2.12/24 unit 2
  vrrp 2
    ip address 192.168.2.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit
bridge 3
  description "AP's 1"
  vlan 300
  ip firewall disable
  ip address 192.168.3.11/24 unit 1
  ip address 192.168.3.12/24 unit 2
  vrrp 3
    ip address 192.168.3.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit
bridge 4
  description "AP's 2"
  vlan 400
  ip firewall disable
  ip address 192.168.4.11/24 unit 1
  ip address 192.168.4.12/24 unit 2
  vrrp 4
    ip address 192.168.4.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit
bridge 5
  description "AP's shared"
  vlan 500
  ip firewall disable
  ip address 192.168.5.11/24 unit 1
  ip address 192.168.5.12/24 unit 2
  vrrp 5
    ip address 192.168.5.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit
bridge 6
  description "Clients"
  vlan 600
  ip firewall disable
  ip address 192.168.6.11/24 unit 1
  ip address 192.168.6.12/24 unit 2
  vrrp 6
    ip address 192.168.6.1/24
    priority 120 unit 1
    priority 110 unit 2
    group 1
    preempt disable
    enable
  exit
  enable
exit

interface gigabitethernet 1/0/1
  description "All data"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 100,300,500,600
exit
interface gigabitethernet 1/0/2
  description "Sync"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 200,400
exit
interface gigabitethernet 2/0/1
  description "All data"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 100,400,500,600
exit
interface gigabitethernet 2/0/2
  description "Sync"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 200,300
exit

tunnel softgre 1
  mode data
  local address 192.168.3.1
  default-profile
  enable
exit

snmp-server
snmp-server community private ro

ip failover
  local-address object-group SYNC_SRC
  remote-address object-group SYNC_DEST
  vrrp-group 1
exit

ip dhcp-server
ip dhcp-server pool ap-vlan-300-pool
  network 192.168.3.0/24
  address-range 192.168.3.50-192.168.3.250
  default-router 192.168.3.1
  dns-server 192.168.3.1
  option 42 ip-address 192.168.3.1
  vendor-specific
    suboption 12 ascii-text "192.168.3.1"
    suboption 15 ascii-text "https://192.168.3.1:8043"
  exit
exit
ip dhcp-server pool users-pool
  network 192.168.6.0/24
  address-range 192.168.6.50-192.168.6.250
  default-router 192.168.6.1
  dns-server 192.168.6.1
exit
ip dhcp-server pool ap-vlan-400-pool
  network 192.168.4.0/24
  address-range 192.168.4.50-192.168.4.250
  default-router 192.168.4.1
  dns-server 192.168.4.1
  option 42 ip-address 192.168.4.1
  vendor-specific
    suboption 12 ascii-text "192.168.3.1"
    suboption 15 ascii-text "https://192.168.3.1:8043"
  exit
exit
ip dhcp-server pool ap-vlan-500-pool
  network 192.168.5.0/24
  address-range 192.168.5.50-192.168.5.250
  default-router 192.168.5.1
  dns-server 192.168.5.1
  option 42 ip-address 192.168.5.1
  vendor-specific
    suboption 12 ascii-text "192.168.3.1"
    suboption 15 ascii-text "https://192.168.3.1:8043"
  exit
exit
ip dhcp-server failover
  mode active-standby
  enable
exit

softgre-controller
  nas-ip-address 127.0.0.1
  failover
  data-tunnel configuration wlc
  aaa radius-profile default_radius
  keepalive-disable
  service-vlan add 600
  enable
exit

wlc
  outside-address 192.168.3.1
  service-activator
    aps join auto
  exit
  airtune
    enable
  exit
  failover
  ap-location default-location
    description "default-location"
    mode tunnel
    ap-profile default-ap
    airtune-profile default_airtune
    ssid-profile levin-ssid
  exit
  airtune-profile default_airtune
    description "default_airtune"
  exit
  ssid-profile levin-ssid
    ssid "levin-ssid"
    radius-profile default-radius
    vlan-id 600
    security-mode WPA2_1X
    band 2g
    band 5g
    enable
  exit
  radio-2g-profile default_2g
    description "default_2g"
  exit
  radio-5g-profile default_5g
    description "default_5g"
  exit
  radio-6g-profile default_6g
    description "default_6g"
  exit
  ap-profile default-ap
    description "default-ap"
    password ascii-text encrypted 8CB5107EA7005AFF
    trace
      networkd enable
      networkd logfile-limit 2000
      hostapd enable
      hostapd logfile-limit 2000
      configd enable
      configd logfile-limit 2000
      netconf enable
      netconf logfile-limit 2000
    exit
    services
      ip ssh server
      ip http server
      lldp-server
    exit
  exit
  radius-profile default-radius
    description "default-radius"
    auth-address 192.168.6.1
    auth-password ascii-text encrypted 8CB5107EA7005AFF
    domain wlc.root
  exit
  wids-profile default-wids
    description "default-wids"
  exit
  ip-pool default-ip-pool
    description "default-ip-pool"
    ap-location default-location
  exit
  enable
exit

ip ssh server

ip tftp client timeout 45
ntp enable
ntp server 192.168.1.20
exit

crypto-sync
  enable
exit

ip http server
ip https server
ip http failover

```


```cfg title="MES"

#Building configuration...
#ISS config ver. 10; SW ver. 10.4.3.4 (67eeccdf) for MES2408PL. Do not remove or edit this line
!   
debug-logging log-path flash:/mnt/  
!
no logging console
!
no spanning-tree
!
vlan 100,200,300-400,500
  vlan active
!                      
username guest password encrypted P6Vt3vbV+ULZ4bn2O+qpew== privilege 1    
!  
mac access-list extended 1 
  deny any any 
!
interface vlan 1
  no ip address
! 
interface vlan 100
  ip address 192.168.1.30 255.255.255.0
! 
interface vlan 300
  ip address 192.168.3.30 255.255.255.0
! 
interface vlan 400
  ip address 192.168.4.30 255.255.255.0
! 
interface vlan 500
  ip address 192.168.5.30 255.255.255.0
! 
interface gigabitethernet 1/0/1
  description "PC"
  switchport general allowed vlan add 100,200,300,400,500
! 
interface gigabitethernet 1/0/2
  description "WLC-1 68:13:E2:7E:82:46 Data"
  switchport general allowed vlan add 100,300
  switchport general pvid 100    
! 
interface gigabitethernet 1/0/3
  description "WLC-2 90:54:B7:3B:A1:40 Data"
  switchport general allowed vlan add 100,400
  switchport general pvid 100    
! 
interface gigabitethernet 1/0/4
  description "WLC-1 68:13:E2:7E:82:46 Sync + Dhcp"
  switchport general allowed vlan add 200,400    
! 
interface gigabitethernet 1/0/5
  description "WLC-2 90:54:B7:3B:A1:40 Sync + Dhcp"
  switchport general allowed vlan add 200,300    
! 
interface gigabitethernet 1/0/6
  description "WEP-30L 90:54:b7:c1:1f:30"
  switchport mode access
  switchport access vlan 300    
! 
interface gigabitethernet 1/0/7
  description "WEP-3ax 68:13:e2:1f:59:80"
  switchport mode access
  switchport access vlan 400    
! 
interface gigabitethernet 1/0/8
  description "WEP-2ac e4:5a:d4:f7:cf:a0"
  switchport mode access
  switchport access vlan 400    
! 
set ip http disable
!  
end

```

