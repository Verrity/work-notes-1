
---
### Теги
#snmp

### [[Home|Назад]]
### Далее
#### [[SNMP-Info]]
---

#### Зависимости ELTEX-WLC-MIB.mib
* ELTEX-SMI-ACTUAL
* elHardware
* eltWlcTraps
#### Первичная настройка

<u>Без настройки MIB</u>
можно не настраивать MIB, тогда в запросе придется указывать `-m /home/<user>/<mib_dir>/ELTEX-WLC-MIB.mib`, но имена OID транслироваться не будут.

**Скачать snmp и дефолтные MIB's**
```bash unfold
sudo apt install snmp snmp-mibs-downloader
snmp download-mibs
```

<u>Настройка MIB</u>
Клонируйте репозиторий [snmp-mibs](https://gitlab.eltex.loc/ems-group/snmp-mibs)
Скоприруйте все MIB кроме `ELTEX-WLC-MIB.mib` из `snmp-mibs/` в `snmp-mibs-storage/` с помощью скрипта
```shell unfold
find snmp-mibs/ -type f -name "*.mib" -exec cp {} snmp-mibs-storage/ \; && rm -f snmp-mibs-storage/ELTEX-WLC-MIB.mib
```

В директориях `/usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf` хранятся дефолтные MIB
В директории `/home/daniil/work/snmp-mibs-storage>` будут храниться прочие MIB
В директории `/home/daniil/work/snmp-mibs/wlc` будет храниться рабочий WLC MIB

**Конфиг `/etc/snmp/snmp.conf` должен выглядеть так:**
```shell unfold
mibs +ALL
mibdirs +/usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf:/home/daniil/work/snmp-mibs-storage:/home/daniil/work/snmp-mibs/wlc
```
Теперь OID будут транслироваться в имена.
#### Как составить snmpwalk
Обычный запрос с OID именем
```bash unfold
snmpwalk -v 2c -c public 192.168.1.1 eltWlcApAllInfoByMacTable 2>/dev/null
```

Обычный запрос с OID числом
```bash unfold
snmpwalk -v 2c -c public 192.168.1.1 .1.3.6.1.4.1.35265.1.224.1.3.2.8 2>/dev/null
```

`2>/dev/null`  - `snmp` выводит огромное количество ворнингов, лучше не видеть `stderr`
Значения ключей можно писать и слитно и раздельно, например: `-cpublic`/`-c public`, `-v2c`/`-v 2c`
Если вы настроили MIB, то можно указывать в качестве OID  имена, так вместо `.1.3.6.1.4.1.35265.1.224.1.3.2.8` можно указать `eltWlcApAllInfoByMacTable`.
Также вместо `.1` в начале OID можно написать `iso`, например `iso.3.6.1.4.1.35265.1.224.1.3.2.8`

Флаги форматирования вывода `-O`:
   * -v - не выводить OID
   * -q - не выводить типы OID
   * -qv - выводить только значения
   * -n - показывать OID в числовом виде

#### MIB валидатор
[MIB Validator](https://snmp.cs.utwente.nl/ietf/mibs/validate/) - выбирайте 6 уровень (максимальный). Первые 4 ошибки есть всегда.
Можно сократить количество ошибок, если загрузить несколько MIB в этом валидаторе [Multiple MIB Validator](https://snmp.cs.utwente.nl/ietf/mibs/validate/upload.php)

























