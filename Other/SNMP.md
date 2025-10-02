
---
### Теги
#snmp

### [[Home|Назад]]
### Далее
#### [[SNMP-Info]]
---

#### Первичная настройка

можно не настраивать MIB, тогда в запросе придется указывать `-m /home/<user>/<mib_dir>/ELTEX-WLC-MIB.mib.`


```bash unfold title="Скачать snmp и дефолтные MIB's"
sudo apt install snmp snmp-mibs-downloader
snmp download-mibs
```

Клонируйте репозиторий [snmp-mibs](https://gitlab.eltex.loc/ems-group/snmp-mibs)

```shell unfold title="/etc/snmp/snmp.conf  <--- добавить /home/daniil/work/snmp-mibs/wlc к остальным директориям"
mibs +ALL
mibdirs +/usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf:/home/<USER>/<SNMP_MIBS_REP_DIR>/wlc
```

#### Как составить snmpwalk

```bash unfold title="Обычный запрос с OID именем"
snmpwalk -v 2c -c public 192.168.1.1 eltWlcApAllInfoByMacTable 2>/dev/null
```


```bash unfold title="Обычный запрос с OID числом"
snmpwalk -v 2c -c public 192.168.1.1 .1.3.6.1.4.1.35265.1.224.1.3.2.8 2>/dev/null
```

`2>/dev/null`  - `snmp` выводит огромное количество ворнингов, лучше не видеть `stderr`
Значения ключей можно писать и слитно и раздельно, например: `-cpublic`/`-c public`, `-v2c`/`-v 2c`
Если вы настроили MIB, то можно указывать в качестве OID  имена, так вместо `.1.3.6.1.4.1.35265.1.224.1.3.2.8` можно указать `eltWlcApAllInfoByMacTable`.
Также вместо `.1` в начале OID можно написать `iso`, например `iso.3.6.1.4.1.35265.1.224.1.3.2.8`

`-O` флаги форматирования вывода:
   * -v - не выводить OID
   * -q - не выводить типы OID
   * -qv - выводить только значения
   * -n - показывать OID в числовом виде

может пригодится `find snmp-mibs/ -type f -name "*.mib" -exec cp {} snmp-mibs-storage/ \;`

























