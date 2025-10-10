
---
### Теги

### [[Home|Назад]]
### Далее
####
---

### Failover (2 WLC)

<u>WLC30 configs:</u>
```cfg fold title="cfg-failover-wlc1"
#!/usr/bin/clish
#303
#1.30.x-1.30.x.b8756a11
#2025-09-22
#12:06:19
hostname WLC-1

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
object-group service netconf
  port-range 830
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
object-group service sync
  port-range 873
exit
object-group service softgre_controller
  port-range 1337
exit

syslog max-files 3
syslog file-size 512
syslog file tmpsys:syslog/default
  severity info
exit
syslog file flash:syslog/radius_log
  severity debug
  match process-name radius-server
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
    user test
      password ascii-text encrypted CDE65039E5591FA3
    exit
  exit
  virtual-server default
    enable
  exit
  enable
exit
username admin
  password encrypted $6$Nyeyr/Lb9igI5Z/z$knxMNdjqKcJs4cWa671000p/tHs9GmigOVa.O4Qz4XZdqvZ0vqx4IoHaZCNz/FF0IaQo4gMOhbCfXpZSI8wgd.
exit

radius-server host 127.0.0.1
  key ascii-text encrypted 8CB5107EA7005AFF
exit
aaa radius-profile default_radius
  radius-server host 127.0.0.1
exit

boot host auto-config
boot host auto-update

vlan 2
  force-up
exit
vlan 3
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
  description "AP pool 1"
  vlan 1
  security-zone trusted
  ip firewall disable
  ip address 192.168.1.2/24
  vrrp id 1
  vrrp ip 192.168.1.1/32
  vrrp priority 120
  vrrp group 1
  vrrp preempt disable
  vrrp timers garp refresh 60
  vrrp
  no spanning-tree
  enable
exit
bridge 2
  vlan 2
  security-zone untrusted
  ip firewall disable
  ip address dhcp
  no spanning-tree
  enable
exit
bridge 3
  description "APClients pool 1"
  vlan 3
  mtu 1458
  security-zone users
  ip firewall disable
  ip address 192.168.2.2/24
  vrrp id 3
  vrrp ip 192.168.2.1/32
  vrrp priority 120
  vrrp group 1
  vrrp preempt disable
  vrrp timers garp refresh 60
  vrrp
  no spanning-tree
  enable
exit

interface gigabitethernet 1/0/1
  mode switchport
  switchport access vlan 2
exit
interface gigabitethernet 1/0/2
  mode switchport
  switchport mode trunk
exit
interface gigabitethernet 1/0/3
  mode switchport
exit
interface gigabitethernet 1/0/4
  mode switchport
exit
interface tengigabitethernet 1/0/1
  mode switchport
  switchport access vlan 2
exit
interface tengigabitethernet 1/0/2
  mode switchport
exit

tunnel softgre 1
  mode data
  local address 192.168.1.1
  default-profile
  enable
exit

ip failover
  local-address 192.168.1.2
  remote-address 192.168.1.3
  vrrp-group 1
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
  rule 11
    action permit
    match protocol vrrp
    enable
  exit
  rule 12
    action permit
    match protocol tcp
    match destination-port object-group softgre_controller
    enable
  exit
  rule 13
    action permit
    match protocol tcp
    match destination-port object-group sync
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
    match destination-port object-group netconf
    enable
  exit
  rule 80
    action permit
    match protocol tcp
    match destination-port object-group sa
    enable
  exit
  rule 90
    action permit
    match protocol udp
    match destination-port object-group radius_auth
    enable
  exit
  rule 100
    action permit
    match protocol gre
    enable
  exit
  rule 110
    action permit
    match protocol tcp
    match destination-port object-group airtune
    enable
  exit
  rule 120
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
  rule 11
    action permit
    match protocol vrrp
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
ip dhcp-server pool ap-pool
  network 192.168.1.0/24
  address-range 192.168.1.2-192.168.1.254
  default-router 192.168.1.1
  dns-server 192.168.1.1
  option 42 ip-address 192.168.1.1
  vendor-specific
    suboption 12 ascii-text "192.168.1.1"
    suboption 15 ascii-text "https://192.168.1.1:8043"
  exit
exit
ip dhcp-server pool users-pool
  network 192.168.2.0/24
  address-range 192.168.2.2-192.168.2.254
  default-router 192.168.2.1
  dns-server 192.168.2.1
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
  service-vlan add 3
  enable
exit

wlc
  outside-address 192.168.1.1
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
    security-mode WPA2_1X
    802.11kv
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
  ap-profile default-ap
    description "default-ap"
    password ascii-text encrypted 8CB5107EA7005AFF
    services
      ip ssh server
      ip http server
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
ntp broadcast-client enable
ntp server 192.168.1.10
exit

crypto-sync
  remote-delete
  enable
exit

ip https server
```

```cfg fold title="cfg-failover-wlc2"
#!/usr/bin/clish
#303
#1.30.x-1.30.x.b8756a11
#2025-09-22
#12:06:19
hostname WLC-2

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
object-group service netconf
  port-range 830
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
object-group service sync
  port-range 873
exit
object-group service softgre_controller
  port-range 1337
exit

syslog max-files 3
syslog file-size 512
syslog file tmpsys:syslog/default
  severity info
exit
syslog file flash:syslog/radius_log
  severity debug
  match process-name radius-server
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
    user test
      password ascii-text encrypted CDE65039E5591FA3
    exit
  exit
  virtual-server default
    enable
  exit
  enable
exit
username admin
  password encrypted $6$Nyeyr/Lb9igI5Z/z$knxMNdjqKcJs4cWa671000p/tHs9GmigOVa.O4Qz4XZdqvZ0vqx4IoHaZCNz/FF0IaQo4gMOhbCfXpZSI8wgd.
exit
radius-server host 127.0.0.1
  key ascii-text encrypted 8CB5107EA7005AFF
exit
aaa radius-profile default_radius
  radius-server host 127.0.0.1
exit

boot host auto-config
boot host auto-update

vlan 2
  force-up
exit
vlan 3
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
  description "AP pool 1"
  vlan 1
  security-zone trusted
  ip firewall disable
  ip address 192.168.1.3/24
  vrrp id 1
  vrrp ip 192.168.1.1/32
  vrrp priority 110
  vrrp group 1
  vrrp preempt disable
  vrrp timers garp refresh 60
  vrrp
  no spanning-tree
  enable
exit
bridge 2
  vlan 2
  security-zone untrusted
  ip firewall disable
  ip address dhcp
  no spanning-tree
  enable
exit
bridge 3
  description "APClients pool 1"
  vlan 3
  mtu 1458
  security-zone users
  ip firewall disable
  ip address 192.168.2.3/24
  vrrp id 3
  vrrp ip 192.168.2.1/32
  vrrp priority 110
  vrrp group 1
  vrrp preempt disable
  vrrp timers garp refresh 60
  vrrp
  no spanning-tree
  enable
exit

interface gigabitethernet 1/0/1
  mode switchport
  switchport access vlan 2
exit
interface gigabitethernet 1/0/2
  mode switchport
  switchport mode trunk
exit
interface gigabitethernet 1/0/3
  mode switchport
exit
interface gigabitethernet 1/0/4
  mode switchport
exit
interface tengigabitethernet 1/0/1
  mode switchport
  switchport access vlan 2
exit
interface tengigabitethernet 1/0/2
  mode switchport
exit

tunnel softgre 1
  mode data
  local address 192.168.1.1
  default-profile
  enable
exit

ip failover
  local-address 192.168.1.3
  remote-address 192.168.1.2
  vrrp-group 1
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
  rule 11
    action permit
    match protocol vrrp
    enable
  exit
  rule 12
    action permit
    match protocol tcp
    match destination-port object-group softgre_controller
    enable
  exit
  rule 13
    action permit
    match protocol tcp
    match destination-port object-group sync
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
    match destination-port object-group netconf
    enable
  exit
  rule 80
    action permit
    match protocol tcp
    match destination-port object-group sa
    enable
  exit
  rule 90
    action permit
    match protocol udp
    match destination-port object-group radius_auth
    enable
  exit
  rule 100
    action permit
    match protocol gre
    enable
  exit
  rule 110
    action permit
    match protocol tcp
    match destination-port object-group airtune
    enable
  exit
  rule 120
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
  rule 11
    action permit
    match protocol vrrp
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
ip dhcp-server pool ap-pool
  network 192.168.1.0/24
  address-range 192.168.1.2-192.168.1.254
  default-router 192.168.1.1
  dns-server 192.168.1.1
  option 42 ip-address 192.168.1.1
  vendor-specific
    suboption 12 ascii-text "192.168.1.1"
    suboption 15 ascii-text "https://192.168.1.1:8043"
  exit
exit
ip dhcp-server pool users-pool
  network 192.168.2.0/24
  address-range 192.168.2.2-192.168.2.254
  default-router 192.168.2.1
  dns-server 192.168.2.1
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
  service-vlan add 3
  enable
exit

wlc
  outside-address 192.168.1.1
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
    security-mode WPA2_1X
    802.11kv
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
  ap-profile default-ap
    description "default-ap"
    password ascii-text encrypted 8CB5107EA7005AFF
    services
      ip ssh server
      ip http server
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
ntp broadcast-client enable
ntp server 192.168.1.10
exit

crypto-sync
  remote-delete
  enable
exit

ip https server
```

<u>MES2408PL config:</u>
```cfg fold title="cfg-failover-mes"

#Building configuration...
!     
!
interface vlan 1 
ip dhcp client vendor-specific MES2408PL
!    
!
interface gigabitethernet 0/1
description "[WLC1] trunc native vlan 1"
no shutdown    
! 
interface gigabitethernet 0/2
description "[WLC2] trunc native vlan 1"
no shutdown    
! 
interface gigabitethernet 0/3
description "[AP] access vlan 1"
no shutdown    
! 
interface gigabitethernet 0/4
description "[] access vlan 1"
no shutdown    
! 
interface gigabitethernet 0/5
description "[AP] access vlan 1"
no shutdown    
! 
interface gigabitethernet 0/6
description "[] access pvid 1"
no shutdown    
! 
interface gigabitethernet 0/7
description "[AP] access pvid 20"
no shutdown    
! 
interface gigabitethernet 0/8
description "[] access vlan 1"
no shutdown    
! 
interface gigabitethernet 0/9
no shutdown    
! 
interface gigabitethernet 0/10
no shutdown    
! 
interface vlan 1
 ip address dhcp
no shutdown
! 
interface vlan 1 
 ipv6 enable   
!
! 
vlan 1
 vlan active
!
!
interface gigabitethernet 0/1
switchport mode trunk
! 
interface gigabitethernet 0/2
switchport mode trunk
! 
interface gigabitethernet 0/3
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
interface gigabitethernet 0/4
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
interface gigabitethernet 0/5
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
interface gigabitethernet 0/6
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
interface gigabitethernet 0/7
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
interface gigabitethernet 0/8
switchport acceptable-frame-type untaggedAndPriorityTagged
switchport mode access
! 
!                                        
set ip http disable
!  
!    

debug-logging flash:/mnt/
!
!  
end
```

<u>Netplan config:</u>
```yaml fold title="5-my-config.yaml"
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    # Интерфейс для корпоративной сети
    enp2s0:
      dhcp4: true
      dhcp6: true
      ipv6-address-generation: "stable-privacy"
    # Интерфейс для локальной сети с устройствами. Управляется bridge "br-wlc".
    enp1s0:
      dhcp4: false
  bridges:
    # Бридж на интерфейс enp1s0 для виртуальных машин и прочего
    br-wlc:
      addresses:
      - "192.168.1.10/24"
      nameservers:
        addresses:
        - 192.168.1.1
      dhcp4: false
      dhcp6: true
      ipv6-address-generation: "stable-privacy"
      interfaces:
      - enp1s0
# vlans:
#   # Подключение ПК
#   br-wlc.1:
#     id: 1
#     link: br-wlc
#     addresses:
#     - "192.168.1.11/24"
#     dhcp4: false
```

### Рабочий SSID
```cfg fold title="cfg-wifi-connect"
#!/usr/bin/clish
#303
#1.30.x-1.30.x.5a891822
#2025-10-02
#12:59:20
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
object-group service netconf
  port-range 830
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
  severity info
exit
syslog file flash:syslog/radius_log
  severity debug
  match process-name radius-server
exit
syslog console
  severity debug
exit
syslog monitor
  severity debug
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
  password encrypted $6$5MFooD1qyj3bqxQQ$SdUSp92BlefV4O6MtcEX0IAPWnjEk2jCngVrWO70LUmqeq0EuwnMdbqV1hCgB27TuvxNONLzmZp5.AgjbmgSJ1
exit
username techsupport
  password encrypted $6$4Of2W/YnG2LPkQ7j$F1Xf0pFt2ex3Q5Yy6S3cZIcN/4CQZ6IF45576QeMhHfWHlBL2tRagcvjJSZPhC3H3LfBoi7FuKN1hgWzNRIVZ/
exit

radius-server host 127.0.0.1
  key ascii-text encrypted 8CB5107EA7005AFF
exit
aaa radius-profile default_radius
  radius-server host 127.0.0.1
exit

tech-support login enable

boot host auto-config
boot host auto-update

vlan 3
  force-up
exit
vlan 2
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
  vlan 1
  security-zone trusted
  ip firewall disable
  ip address 192.168.1.1/24
  no spanning-tree
  enable
exit
bridge 2
  vlan 2
  security-zone untrusted
  ip firewall disable
  ip address dhcp
  no spanning-tree
  enable
exit
bridge 3
  vlan 3
  mtu 1458
  security-zone users
  ip firewall disable
  ip address 192.168.2.1/24
  no spanning-tree
  enable
exit

interface gigabitethernet 1/0/1
  mode switchport
exit
interface gigabitethernet 1/0/2
  mode switchport
exit
interface gigabitethernet 1/0/3
  mode switchport
exit
interface gigabitethernet 1/0/4
  mode switchport
exit
interface tengigabitethernet 1/0/1
  mode switchport
  switchport access vlan 2
exit
interface tengigabitethernet 1/0/2
  mode switchport
exit

tunnel softgre 1
  mode data
  local address 192.168.1.1
  default-profile
  enable
exit

snmp-server
snmp-server community private1 rw
snmp-server community private rw
snmp-server community public ro

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
    match destination-port object-group netconf
    enable
  exit
  rule 80
    action permit
    match protocol tcp
    match destination-port object-group sa
    enable
  exit
  rule 90
    action permit
    match protocol udp
    match destination-port object-group radius_auth
    enable
  exit
  rule 100
    action permit
    match protocol gre
    enable
  exit
  rule 110
    action permit
    match protocol tcp
    match destination-port object-group airtune
    enable
  exit
  rule 120
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
ip dhcp-server pool ap-pool
  network 192.168.1.0/24
  address-range 192.168.1.2-192.168.1.254
  default-router 192.168.1.1
  dns-server 192.168.1.1
  option 42 ip-address 192.168.1.1
  vendor-specific
    suboption 12 ascii-text "192.168.1.1"
    suboption 15 ascii-text "https://192.168.1.1:8043"
  exit
exit
ip dhcp-server pool users-pool
  network 192.168.2.0/24
  address-range 192.168.2.2-192.168.2.254
  default-router 192.168.2.1
  dns-server 192.168.2.1
exit

softgre-controller
  nas-ip-address 127.0.0.1
  data-tunnel configuration wlc
  aaa radius-profile default_radius
  keepalive-disable
  service-vlan add 3
  enable
exit

wlc
  outside-address 192.168.1.1
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
    ssid "LEVIN_TEST_SSID"
    radius-profile default-radius
    vlan-id 3
    security-mode WPA2_1X
    802.11kv
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
  ap-profile default-ap
    description "default-ap"
    password ascii-text encrypted 8CB5107EA7005AFF
    services
      ip ssh server
      ip http server
    exit
  exit
  radius-profile default-radius
    description "default-radius"
    auth-address 192.168.1.1
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
ntp broadcast-client enable
ntp server 192.168.1.10
exit

ip https server
```
