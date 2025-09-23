
---
### Теги

### Назад
#### [[Home]]
### Далее
#### [[Скрипты bash в home-local-bin]]
####  [[Сетевые настройки]]
---

### Сеть
```yaml folded title="/etc/netplan/10-my-config.yaml"
nnetwork:
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
      match:
        macaddress: "8C:90:2D:FD:AB:9D"
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
  vlans:
    # Подключение ПК
    br-wlc.10:
      id: 10
      link: br-wlc
    # Подключение ТД
    br-wlc.20:
      id: 20
      link: br-wlc
    # Подключение пользователей ТД
    br-wlc.30:
      id: 30
      link: br-wlc
    # Синхронизация
    br-wlc.40:
      id: 40
      link: br-wlc
```

```bash folded title="Настроить права доступа чтобы netplan не ругался"
sudo chmod 600 /etc/netplan/10-my-config.yaml
sudo chown root:root /etc/netplan/10-my-config.yaml
```
C



### DHCP
Конфиг:
```shell folded title="/etc/dhcp/dhcpd.conf"
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#

# option definitions common to all supported networks...
#option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
#log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

#subnet 192.168.176.0 netmask 255.255.252.0 {
#  range 192.168.176.160 192.168.176.170;
#  option routers 192.168.176.1;
#  option domain-name-servers 8.8.8.8;
#}


# ============================= 43 option WLC

# !!!!!!!!!!!!!!!!!!!!!!!!!
#      Test Docker
# http ... 192.168.1.2 ...

subnet 192.168.1.0 netmask 255.255.255.0 {
  option routers 192.168.1.2;
  range 192.168.1.3 192.168.1.10;
  # 192.168.1.2
  # 0F:18 + https://192.168.1.2:8043
  option vendor-encapsulated-options 0F:18:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:2E:32:3A:38:30:34:33;
}

# ---------
#      Default this is
# http ... 192.168.1.1 ...

# subnet 192.168.1.0 netmask 255.255.255.0 {
#   option routers 192.168.1.1;
#   range 192.168.1.3 192.168.1.10;
#   option vendor-encapsulated-options 0F:18:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:2E:31:3A:38:30:34:33;
# }

# =============================

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.example.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.

#host wep12-ac {
#  hardware ethernet a8:f9:4b:b4:97:c0;
#  fixed-address 192.168.176.151;
#}

#host wep-3ax {
#  hardware ethernet cc:9d:a2:c2:6e:40;
#  hardware ethernet 68:13:e2:02:50:e0;
#  fixed-address 192.168.176.152;
#  option vendor-encapsulated-options 0F:1C:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:37:36:2E:31:35:30:3A:38:30:34:33;
  #option vendor-encapsulated-options 0F:1C:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:37:36:2E:31:35:30:3A:38:30:34:33:11:01:04;
#}

#host wep-3ax {
#  hardware ethernet cc:9d:a2:c2:6e:40;
#  fixed-address 192.168.1.2;
#  option vendor-encapsulated-options 0F:18:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:2E:31:3A:38:30:34:33; #https://192.168.1.1:8043
#}

#host wep-3ax {
#  hardware ethernet 00:90:4c:1e:20:00;
#  fixed-address 192.168.176.152;
#}

#host wep2-ac_1 {
#  hardware ethernet a8:f9:4b:b5:fb:a0;
#  fixed-address 192.168.176.153;
#}

#host wep2-ac_2 {
#  hardware ethernet a8:f9:4b:b4:b3:00;
#  fixed-address 192.168.176.154;
#}

#host wop-2l {
#  hardware ethernet 94:08:53:2c:0a:31;
#  fixed-address 192.168.176.155;
#}

#host iphone {
#  hardware ethernet c8:b1:cd:43:cf:94;
#  fixed-address 192.168.176.156;
#}

#host iphone2 {
 # hardware ethernet 34:fe:77:8f:4f:89;
#  fixed-address 192.168.176.156;
#}

#host client_2 {
#  hardware ethernet b4:3a:28:31:34:1b;
#  fixed-address 192.168.176.157;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}

```

### TFTP
```shell folded title="/etc/default/tftp-hpa"
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS="192.168.1.2:69"
TFTP_OPTIONS="--secure --create"
```
Вот тут нужно поебаться с правами доступа. И обязательно сделать `chown` на `tftp`, подробнее гугли.
### Vim
```vim folded title="~/.vimrc"
set clipboard+=unnamedplus
set expandtab
set smarttab
set tabstop=2
set softtabstop=2
set shiftwidth=2

set number
set foldcolumn=0
set mouse=a

colorscheme desert

vnoremap <C-c> "+y
vnoremap <C-v> "+p

set ignorecase
set smartcase
set hlsearch
set incsearch
```
### .bashrc
```shell folded title="~/.bashrc additionals"
# Show a current active git branch in the shell prompt
export PS1='\[\033[01;34m\] \w\[\033[01;33m\]$(__git_ps1)\[\033[01;34m\] \$\[\033[00m\] ' 

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
```

### VM запустить из командной строки
```bash folded title="Запустить виртуальную машину"
virsh --connect qemu:///system start "ubuntu-22-04-lts-server"
```
```bash folded title="Запустить UI приложение"
virt-manager --connect qemu:///system --show-domain-console "ubuntu-22-04-lts-server"
```

#### NTP server
[Инструкция по установке](https://www.dmosk.ru/miniinstruktions.php?mini=ntp-server-ubuntu)