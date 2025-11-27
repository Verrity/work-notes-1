
---
### Теги

### [[ESR|Назад]]
### Далее
####
---
#### Загрузить прошивку через u-boot
Если ни одной прошивки на устройстве не осталось, и нельзя выйти из `uboot`, то вот решение
```u-boot unfold
ipaddr 192.168.1.1
serverip 192.168.1.10
firmware_file wlc30.firmware
saveenv
netboot
```
`ipaddr` - адрес `wlc`; `serverip` - адрес `tftp` сервера ; `firmware_file` - имя файла на сервере ; `saveenv` - ?; `netboot` - загрузить прошивку и запустить устройство

#### Очистить что-то
```unfold
clear_mtd_data
clear_mtd_config
```
clear mtd data и clear mtd config