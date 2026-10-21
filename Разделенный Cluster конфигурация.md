
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
snmp-server community public ro

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
clock timezone gmt +7

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

```shell title="clspa.sh - скрипт для быстрой настройки и возвращения clspa/default сетевой конфигурации"
#!/bin/bash

# ============================================
# КОНФИГУРАЦИЯ
# ============================================
TAP1="clspa-tap-vm1"
TAP2="clspa-tap-vm2"
ETH="enp1s0"
BR="clspa-br-wlc"
WAN_BR="clspa-br-wan"
WAN_ETH="enp2s0"

VLAN_IDS=(100 200 300 400 500)
declare -A VLAN_IP
VLAN_IP[100]="192.168.1.20/24"
VLAN_IP[200]="192.168.2.20/24"
VLAN_IP[300]="192.168.3.20/24"
VLAN_IP[400]="192.168.4.20/24"
VLAN_IP[500]="192.168.5.20/24"

NETPLAN_CONFIG_DIR_NAME="config-storage"
NETPLAN_CURRENT_CONFIG_LINK_NAME="99-netplan-link.yaml"
NETPLAN_RESTORE_CONFIG_NAME="01-networkd-default.yaml"
NETPLAN_CLSPA_CONFIG_NAME="clspa-networkd-config.yaml"

# ======================================
# ФУНКЦИИ ВЫВОДА
# ======================================

print_info() {
    echo "  $*"
}

print_step() {
    local -n _step="$1"
    shift 1
    local message="$*"
    ((_step++))
    echo "[$_step] $message"
}

print_start() {
    echo "=========================================="
    echo "$*"
    echo "=========================================="
    echo ""
}

print_end() {
    echo "=========================================="
    echo "ВЫПОЛНЕНИЕ ЗАВЕРШЕНО"
    echo "=========================================="
    echo ""
}

# ======================================
# ФУНКЦИЯ ПРОВЕРКИ ПИНГА
# ======================================

ping_check() {
    local ip="$1"
    local desc="${2:-$ip}"
    print_info "Проверка доступности хоста $desc (IP: $ip) с помощью двух ICMP-запросов..."
    if ping -c 2 -W 2 "$ip" >/dev/null 2>&1; then
        print_info "  Результат: хост $desc (IP: $ip) ДОСТУПЕН"
        return 0
    else
        print_info "  Результат: хост $desc (IP: $ip) НЕДОСТУПЕН"
        return 1
    fi
}

# ======================================
# ФУНКЦИИ УПРАВЛЕНИЯ NETPLAN
# ======================================

netplan_stop() {
    print_info "Остановка systemd-networkd и его сокета..."
    sudo systemctl stop systemd-networkd 2>/dev/null || true
    sudo systemctl stop systemd-networkd.socket 2>/dev/null || true

    print_info "Убийство забытых dhclient, перезаписывающих статические IP..."
    sudo pkill -x dhclient 2>/dev/null || true
}

netplan_start() {
    sudo rm -f /run/systemd/netif/links/* 2>/dev/null || true
    sudo rm -f /run/systemd/resolve/* 2>/dev/null || true

    print_info "Запуск systemd-networkd и его сокета для применения конфигурации netplan..."
    sudo systemctl start systemd-networkd 2>/dev/null || true
    sudo systemctl start systemd-networkd.socket 2>/dev/null || true
}

netplan_fix_permissions() {
    local config_file="$1"
    print_info "Установка прав доступа 600 и владельца root:root для файла $config_file..."
    sudo chmod 600 "$config_file" 2>/dev/null || true
    sudo chown root:root "$config_file" 2>/dev/null || true
}

netplan_set_config() {
    local config_file="$1"
    local config_name="$2"

    print_info "Активация конфигурации netplan с именем '$config_name' (файл: $config_file)..."

    if [ -L "/etc/netplan/$NETPLAN_CURRENT_CONFIG_LINK_NAME" ]; then
        sudo rm -f "/etc/netplan/$NETPLAN_CURRENT_CONFIG_LINK_NAME"
        print_info "  Удалена старая символическая ссылка /etc/netplan/$NETPLAN_CURRENT_CONFIG_LINK_NAME"
    fi

    if [ ! -f "$config_file" ]; then
        print_info "  ОШИБКА: файл конфигурации $config_file не найден. Активация невозможна."
        return 1
    fi

    netplan_fix_permissions "$config_file"
    sudo ln -s "$config_file" "/etc/netplan/$NETPLAN_CURRENT_CONFIG_LINK_NAME"
    print_info "  Создана символическая ссылка: /etc/netplan/$NETPLAN_CURRENT_CONFIG_LINK_NAME -> $config_file"

    print_info "  Генерация и применение netplan (netplan generate и netplan apply)..."
    sudo netplan generate 2>/dev/null || true
    sudo netplan apply 2>/dev/null || {
        print_info "  ПРЕДУПРЕЖДЕНИЕ: netplan apply завершился с ошибкой. Выполняется перезапуск systemd-networkd вручную..."
        sudo systemctl restart systemd-networkd 2>/dev/null || true
    }

    if systemctl is-active --quiet systemd-networkd; then
        print_info "  systemd-networkd успешно запущен и работает"
    else
        print_info "  ПРЕДУПРЕЖДЕНИЕ: systemd-networkd не запущен. Выполняется запуск вручную..."
        sudo systemctl start systemd-networkd 2>/dev/null || true
    fi

    return 0
}

netplan_setup_clspa() {
    print_info "Создание конфигурационного файла netplan для режима CLSPA..."

    local clspa_netplan_path="/etc/netplan/$NETPLAN_CONFIG_DIR_NAME/$NETPLAN_CLSPA_CONFIG_NAME"
    sudo mkdir -p "/etc/netplan/$NETPLAN_CONFIG_DIR_NAME"
    print_info "  Создана директория /etc/netplan/$NETPLAN_CONFIG_DIR_NAME (если её не было)"

    sudo tee "$clspa_netplan_path" > /dev/null << EOF
network:
  version: 2
  renderer: networkd
  ethernets:
    $WAN_ETH:
      dhcp4: true
EOF

    print_info "  Файл конфигурации создан: $clspa_netplan_path"
    netplan_set_config "$clspa_netplan_path" "CLSPA"
}

netplan_restore_default() {
    print_info "Восстановление конфигурации netplan по умолчанию..."
    local default_config="/etc/netplan/$NETPLAN_CONFIG_DIR_NAME/$NETPLAN_RESTORE_CONFIG_NAME"

    if [ ! -f "$default_config" ]; then
        print_info "  ОШИБКА: файл $default_config не найден. Восстановление невозможно."
        return 1
    fi

    netplan_set_config "$default_config" "DEFAULT"
}

# ======================================
# ОСНОВНЫЕ ФУНКЦИИ НАСТРОЙКИ И ОЧИСТКИ
# ======================================

network_clspa_cleanup() {
    local step=0
    local name="Удаление созданных сетевых интерфейсов (bridge, TAP, VLAN-интерфейсы)"
    print_start "$name"

    print_step step "Отвязывание физического интерфейса $ETH и TAP-интерфейсов $TAP1, $TAP2 от bridge $BR..."
    sudo ip link set "$ETH" nomaster 2>/dev/null || true
    sudo ip link set "$TAP1" nomaster 2>/dev/null || true
    sudo ip link set "$TAP2" nomaster 2>/dev/null || true

    print_step step "Удаление всех IP-адресов с интерфейса $ETH..."
    sudo ip addr flush dev "$ETH" 2>/dev/null

    print_step step "Отключение bridge $BR (перевод в состояние DOWN)..."
    sudo ip link set "$BR" down 2>/dev/null || true

    print_step step "Удаление VLAN-интерфейсов (vlan100..vlan500), созданных поверх bridge..."
    for vid in "${VLAN_IDS[@]}"; do
        sudo ip link delete "vlan${vid}" 2>/dev/null && echo "  vlan${vid} удалён"
    done

    print_step step "Удаление bridge $BR и TAP-интерфейсов $TAP1, $TAP2..."
    sudo ip link delete "$BR" 2>/dev/null && echo "  Bridge $BR удалён" || echo "  Bridge $BR не существовал"
    sudo ip link delete "$TAP1" 2>/dev/null && echo "  $TAP1 удалён" || echo "  $TAP1 не существовал"
    sudo ip link delete "$TAP2" 2>/dev/null && echo "  $TAP2 удалён" || echo "  $TAP2 не существовал"

    print_end
}

network_clspa_setup() {
    local step=0
    local name="Настройка bridge $BR и VLAN-интерфейсов для CLSPA"
    print_start "$name"

    print_step step "Создание bridge $BR и его поднятие (UP)..."
    sudo ip link add "$BR" type bridge
    sudo ip link set "$BR" up
    sudo ip link set dev "$BR" type bridge vlan_filtering 1

    print_step step "Настройка bridge $BR как VLAN-aware (добавление всех VLAN из списка и установка PVID=100)..."
    for vid in "${VLAN_IDS[@]}"; do
        sudo bridge vlan add dev "$BR" vid "$vid" self
    done
    sudo bridge vlan add dev "$BR" vid 100 pvid self

    print_step step "Добавление физического интерфейса $ETH в bridge $BR в качестве trunk-порта..."
    sudo ip link set "$ETH" master "$BR"

    print_step step "Удаление старых vlan и добавление VLAN для $ETH в bridge $BR..."
    sudo bridge vlan delete dev $ETH vid 1 2>/dev/null || true
    for vid in "${VLAN_IDS[@]}"; do
        sudo bridge vlan delete dev $ETH vid $vid 2>/dev/null || true
        sudo bridge vlan add dev $ETH vid $vid
    done

    print_step step "Создание TAP-интерфейсов $TAP1 и $TAP2 с поднятием (UP)..."
    sudo ip tuntap add "$TAP1" mode tap
    sudo ip tuntap add "$TAP2" mode tap
    sudo ip link set "$TAP1" up
    sudo ip link set "$TAP2" up

    print_step step "Привязка TAP-интерфейсов к bridge и назначение PVID (untagged) для VLAN 300 и 400 соответственно..."
    sudo ip link set "$TAP1" master "$BR"
    sudo ip link set "$TAP2" master "$BR"
    sudo bridge vlan add dev "$TAP1" vid 300 pvid untagged
    sudo bridge vlan add dev "$TAP2" vid 400 pvid untagged

    print_step step "Создание VLAN-интерфейсов поверх bridge и назначение IP-адресов согласно конфигурации..."
    for vid in "${VLAN_IDS[@]}"; do
        local vlan_if="vlan${vid}"
        sudo ip link add link "$BR" name "$vlan_if" type vlan id "$vid"
        sudo ip link set "$vlan_if" up
        sudo ip addr add "${VLAN_IP[$vid]}" dev "$vlan_if"
        print_info "  $vlan_if -> ${VLAN_IP[$vid]}"
    done

    print_step step "Отображение текущего состояния bridge (список портов) и VLAN..."
    print_info "  Список портов bridge:"
    bridge link show
    print_info "  Таблица VLAN на bridge:"
    bridge vlan show
    print_info "  Назначенные IP-адреса на VLAN-интерфейсах:"
    for vid in "${VLAN_IDS[@]}"; do
        ip a show "vlan${vid}" | grep inet
    done

    print_end "$name"
}

network_clspa_check() {
    local step=0
    local name="Проверка сетевой доступности локальных IP и удалённых коммутаторов"
    print_start "$name"

    print_step step "Проверка доступности локальных VLAN-интерфейсов (собственные IP)..."
    for vid in "${VLAN_IDS[@]}"; do
        local ip=$(echo "${VLAN_IP[$vid]}" | cut -d/ -f1)
        ping_check "$ip" "VLAN $vid"
    done

    print_step step "Проверка доступности удалённых коммутаторов (предполагаемый адрес .30 в каждой VLAN)..."
    print_info "  Примечание: проверка выполняется только если коммутатор настроен и доступен."
    for vid in "${VLAN_IDS[@]}"; do
        local base=$(echo "${VLAN_IP[$vid]}" | cut -d. -f1-3)
        local switch_ip="${base}.30"
        ping_check "$switch_ip" "коммутатор в VLAN $vid"
    done

    print_end "$name"
}

# ======================================
# ГЛАВНАЯ ЛОГИКА СКРИПТА
# ======================================

case "$1" in
    --cleanup)
        network_clspa_cleanup
        ;;

    --restore)
        netplan_stop
        network_clspa_cleanup
        netplan_restore_default
        netplan_start
        ;;

    --setup)
        netplan_stop
        network_clspa_cleanup
        network_clspa_setup
        netplan_setup_clspa
        netplan_start
        ;;

    *)
        echo "Использование:"
        echo "  $0 --cleanup              - остановка + очистка + запуск"
        echo "  $0 --restore              - остановка + очистка + восстановление DEFAULT + запуск"
        echo "  $0 --setup                - остановка + очистка + установка CLSPA + запуск"
        exit 1
        ;;
esac
```