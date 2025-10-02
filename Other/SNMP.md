
---
### Теги
#snmp

### [[Home|Назад]]
### Далее
#### [[SNMP-Info]]
---

```
snmpwalk -v 2c -c public 192.168.1.1 .1.3.6.1.4.1.35265.1.224.1.3.2.8
snmpwalk -v2c -c public -O n 192.168.1.1 eltWlcApAllInfoByMacTable 2>/dev/null
```

`2>/dev/null`  - `snmp` выводит огромное количество ворнингов, лучше не видеть `stderr`
Значения

-O флаги
   -v - не выводить OID
   -q - не выводить типы OID
   -qv - выводить только значения
   -n - показывать OID в числовом виде



























