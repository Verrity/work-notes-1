
---
### Теги

### [[Конфигурации WLC|Назад]]
### Далее
####
---


```cfg title="WLC"
 #!/usr/bin/clish
#390
#1.39.x
#2026-06-04
#16:52:56
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

object-group service ssh
  port-range 22
exit
object-group service dhcp_server
  port-range 67
exit
object-group service dhcp_client
  port-range 68
exit
object-group service ntp
  port-range 123
exit
object-group service dns
  port-range 53
exit
object-group service radius_auth
  port-range 1812
exit
object-group service sa
  port-range 8043-8044
exit
object-group service airtune
  port-range 8099
exit
object-group service web
  port-range 443
exit

syslog max-files 3
syslog file-size 512
syslog file tmpsys:syslog/default
  severity debug
exit
syslog file flash:syslog/radius_log
  severity debug
  match process-name radius-server
exit
syslog file flash:syslog/default
  severity info
exit
syslog console
  severity warning
exit

logging radius

radius-server local
  nas ap
    key ascii-text encrypted 8CB5107EA7005AFF
    network 192.168.1.0/24
  exit
  nas local
    key ascii-text encrypted 8CB5107EA7005AFF
    network 127.0.0.1/32
  exit
  domain default
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

vlan 1000
  force-up
exit
vlan 2000
  force-up
exit
vlan 3001
  force-up
exit
vlan 3002
  force-up
exit
vlan 4000
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
  vlan 1000
  ip firewall disable
  ip address 192.168.1.11/24 unit 1
  ip address 192.168.1.12/24 unit 2
  vrrp 1
    ip address 192.168.1.1/24
    priority 120 unit 1
    group 1
    enable
  exit
  enable
exit
bridge 2
  description "SYNC"
  vlan 2000
  ip firewall disable
  ip address 192.168.2.11/24 unit 1
  ip address 192.168.2.12/24 unit 2
  vrrp 2
    ip address 192.168.2.1/24
    priority 120 unit 1
    group 1
    enable
  exit
  enable
exit
bridge 3
  description "AP's 1"
  vlan 3001
  ip firewall disable
  ip address 192.168.3.11/24 unit 1
  ip address 192.168.3.12/24 unit 2
  vrrp 3
    ip address 192.168.3.1/24
    priority 120 unit 1
    group 1
    enable
  exit
  enable
exit
bridge 4
  description "AP's 2"
  vlan 3002
  ip firewall disable
  ip address 192.168.4.11/24 unit 1
  ip address 192.168.4.12/24 unit 2
  vrrp 4
    ip address 192.168.4.1/24
    priority 120 unit 1
    group 1
    enable
  exit
  enable
exit
bridge 5
  description "Clients"
  vlan 4000
  ip firewall disable
  ip address 192.168.5.11/24 unit 1
  ip address 192.168.5.12/24 unit 2
  vrrp 5
    ip address 192.168.5.1/24
    priority 120 unit 1
    group 1
    enable
  exit
  enable
exit

interface gigabitethernet 1/0/1
  description "All data"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 1000,3001,4000
exit
interface gigabitethernet 1/0/2
  description "Sync"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 2000,3002
exit
interface gigabitethernet 2/0/1
  description "All data"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 1000,3001,4000
exit
interface gigabitethernet 2/0/2
  description "Sync"
  mode switchport
  switchport mode trunk
  switchport trunk allowed vlan add 2000,3001
exit

tunnel softgre 1
  mode data
  local address 192.168.1.1
  default-profile
  enable
exit

security zone-pair trusted untrusted
  rule 1
    action permit
    enable
  exit
exit
security zone-pair trusted trusted
  rule 1
    action permit
    enable
  exit
exit
security zone-pair trusted self
  rule 10
    action permit
    match protocol tcp
    match destination-port object-group ssh
    enable
  exit
  rule 20
    action permit
    match protocol icmp
    enable
  exit
  rule 30
    action permit
    match protocol udp
    match source-port object-group dhcp_client
    match destination-port object-group dhcp_server
    enable
  exit
  rule 40
    action permit
    match protocol udp
    match destination-port object-group ntp
    enable
  exit
  rule 50
    action permit
    match protocol tcp
    match destination-port object-group dns
    enable
  exit
  rule 60
    action permit
    match protocol udp
    match destination-port object-group dns
    enable
  exit
  rule 70
    action permit
    match protocol tcp
    match destination-port object-group sa
    enable
  exit
  rule 80
    action permit
    match protocol udp
    match destination-port object-group radius_auth
    enable
  exit
  rule 90
    action permit
    match protocol gre
    enable
  exit
  rule 100
    action permit
    match protocol tcp
    match destination-port object-group airtune
    enable
  exit
  rule 110
    action permit
    match protocol tcp
    match destination-port object-group web
    enable
  exit
exit
security zone-pair untrusted self
  rule 1
    action permit
    match protocol udp
    match source-port object-group dhcp_server
    match destination-port object-group dhcp_client
    enable
  exit
exit
security zone-pair users self
  rule 10
    action permit
    match protocol icmp
    enable
  exit
  rule 20
    action permit
    match protocol udp
    match source-port object-group dhcp_client
    match destination-port object-group dhcp_server
    enable
  exit
  rule 30
    action permit
    match protocol tcp
    match destination-port object-group dns
    enable
  exit
  rule 40
    action permit
    match protocol udp
    match destination-port object-group dns
    enable
  exit
exit
security zone-pair users untrusted
  rule 1
    action permit
    enable
  exit
exit

security passwords default-expired

nat source
  ruleset factory
    to zone untrusted
    rule 10
      description "replace 'source ip' by outgoing interface ip address"
      action source-nat interface
      enable
    exit
  exit
exit

ip dhcp-server
ip dhcp-server pool ap-vlan-3001-pool
  network 192.168.3.0/24
  address-range 192.168.3.20-192.168.3.254
  default-router 192.168.3.1
  dns-server 192.168.3.1
  option 42 ip-address 192.168.3.1
  vendor-specific
    suboption 12 ascii-text "192.168.3.1"
    suboption 15 ascii-text "https://192.168.3.1:8043"
  exit
exit
ip dhcp-server pool users-pool
  network 192.168.5.0/24
  address-range 192.168.5.20-192.168.5.254
  default-router 192.168.5.1
  dns-server 192.168.5.1
exit
ip dhcp-server pool ap-vlan-3002-pool
  network 192.168.4.0/24
  address-range 192.168.4.20-192.168.4.254
  default-router 192.168.4.1
  dns-server 192.168.4.1
  option 42 ip-address 192.168.4.1
  vendor-specific
    suboption 12 ascii-text "192.168.3.1"
    suboption 15 ascii-text "https://192.168.3.1:8043"
  exit
exit

softgre-controller
  nas-ip-address 127.0.0.1
  data-tunnel configuration wlc
  aaa radius-profile default_radius
  keepalive-disable
  service-vlan add 4000
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
  ap-location default-location
    description "default-location"
    mode tunnel
    ap-profile default-ap
    airtune-profile default_airtune
    ssid-profile default-ssid
  exit
  airtune-profile default_airtune
    description "default_airtune"
  exit
  ssid-profile default-ssid
    description "default-ssid"
    ssid "default-ssid"
    radius-profile default-radius
    vlan-id 3
    security-mode WPA2_WPA3_1X
    mfp-mode capable
    802.11kv
    band 2g
    band 5g
    band 6g
    enable
  exit
  radio-2g-profile default_2g
    description "default_2g"
    tx-power minimal
    tx-power-max minimal
  exit
  radio-5g-profile default_5g
    description "default_5g"
    tx-power minimal
    tx-power-max minimal
  exit
  radio-6g-profile default_6g
    description "default_6g"
    tx-power minimal
    tx-power-max minimal
  exit
  ap-profile default-ap
    description "default-ap"
    password ascii-text encrypted 8CB5107EA7005AFF
    services
      ip ssh server
      ip http server
      lldp-server
    exit
  exit
  radius-profile default-radius
    description "default-radius"
    auth-address 192.168.1.1
    auth-password ascii-text encrypted 8CB5107EA7005AFF
    domain default
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
ntp server 192.168.1.111
exit

ip http server
ip https server
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
vlan 1000,2000,3001-3002,4000
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
interface vlan 1000
  ip address 192.168.1.250 255.255.255.0
!
interface vlan 3001
  ip address 192.168.3.250 255.255.255.0
!
interface vlan 3002
  ip address 192.168.4.250 255.255.255.0
!
interface vlan 4000
  ip address 192.168.5.250 255.255.255.0
!
interface gigabitethernet 1/0/1
  description "PC"
  switchport mode access
  switchport access vlan 1000
!
interface gigabitethernet 1/0/2
  description "WLC-1 68:13:E2:7E:82:46 Data"
  switchport general allowed vlan add 1000,3001
  switchport general pvid 1000
!
interface gigabitethernet 1/0/3
  description "WLC-2 90:54:B7:3B:A1:40 Data"
  switchport general allowed vlan add 1000,3002
  switchport general pvid 1000
!
interface gigabitethernet 1/0/4
  description "WLC-1 68:13:E2:7E:82:46 Sync"
  switchport general allowed vlan add 2000,3002
!
interface gigabitethernet 1/0/5
  description "WLC-2 90:54:B7:3B:A1:40 Sync"
  switchport general allowed vlan add 2000,3001
!
interface gigabitethernet 1/0/6
  description "WEP-30L 90:54:b7:c1:1f:30"
  switchport mode access
  switchport access vlan 3001
!
interface gigabitethernet 1/0/7
  description "WEP-3ax 68:13:e2:1f:59:80"
  switchport mode access
  switchport access vlan 3002
!
interface gigabitethernet 1/0/8
  description "WEP-2ac e4:5a:d4:f7:cf:a0"
  switchport mode access
  switchport access vlan 3002
!
set ip http disable
!
end

```

