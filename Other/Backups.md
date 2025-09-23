
---
### Теги

### Назад
### Далее
####
---

### Netplan
```yaml folded title="90-NM-ae9132e0-fcf6-3d96-9f77-aa48efd55419.yaml" 
network:
  version: 2
  ethernets:
    NM-ae9132e0-fcf6-3d96-9f77-aa48efd55419:
      renderer: NetworkManager
      match:
        name: "enp2s0"
      dhcp4: true
      dhcp6: true
      ipv6-address-generation: "stable-privacy"
      wakeonlan: true
      networkmanager:
        uuid: "ae9132e0-fcf6-3d96-9f77-aa48efd55419"
        name: "Wired connection 1"
        passthrough:
          connection.autoconnect-priority: "-999"
          connection.timestamp: "1758074375"
          ethernet._: ""
          ipv6.ip6-privacy: "-1"
          proxy._: ""
```
