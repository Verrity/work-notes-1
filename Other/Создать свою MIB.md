1. Придумать свой enterprise OID организации (допустим 9999)
```text unfold
1.3.6.1.4.1.9999
1.3.6.1.4.1 - enterprises
```

```MIB title="Простейший MIB (simple_mib.mib)"
MY-TEST-MIB DEFINITIONS ::= BEGIN

IMPORTS
    OBJECT-TYPE, Integer32, DisplayString
        FROM SNMPv2-SMI;

myTestMib MODULE-IDENTITY
    LAST-UPDATED "202508210000Z"
    ORGANIZATION "My Test Organization"
    CONTACT-INFO "me@example.com"
    DESCRIPTION "Простая тестовая MIB."
    ::= { enterprises 9999 }

-- Корень для моих объектов
myObjects OBJECT IDENTIFIER ::= { myTestMib 1 }

-- Тестовая строка
myString OBJECT-TYPE
    SYNTAX      DisplayString (SIZE (0..64))
    MAX-ACCESS  read-write
    STATUS      current
    DESCRIPTION "Просто строка для теста."
    ::= { myObjects 1 }

-- Тестовый счётчик
myCounter OBJECT-TYPE
    SYNTAX      Integer32 (0..1000)
    MAX-ACCESS  read-only
    STATUS      current
    DESCRIPTION "Счётчик запросов."
    ::= { myObjects 2 }

END
```
Здесь:
* `1.3.6.1.4.1.9999.1.1` - строка
* `1.3.6.1.4.1.9999.1.2` - счетчик

Скопировать MIB в каталог **Net-SNMP**:
`/usr/share/snmp/mibs/` или `/usr/share/snmp/mibs/`
Но нужно убедиться, что в `snmp.conf`есть:
```ruby
mibdirs +/usr/share/snmp/mibs
mibs +MY-
```


