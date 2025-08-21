
---
### Теги
#info #snmp
### Назад
#### [[SNMP-Info]]
### Далее
---

1. Придумать свой enterprise OID организации (допустим 9999)
```text unfold
1.3.6.1.4.1.9999
1.3.6.1.4.1 - enterprises
```

```MIB title="Создать простейший MIB (my_test_mib.mib)"
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

Скопировать MIB в каталог **Net-SNMP**: `/usr/share/snmp/mibs/` или `/usr/share/snmp/mibs/`
```ruby unfold title:"Но нужно убедиться, что в snmp.conf есть:"
mibdirs +/usr/share/snmp/mibs
mibs +MY-TEST-MIB
```

Теперь нужно запустить тестовый SNMP - агент с данной MIB
Есть 2 варианта:
1. <u>Использовать `snmpd` + extend</u>
Net-SNMP `snmpd` умеет расширяться:
`pass`/`extend`/`exec` позволяют привязать свои OID с скриптам.
```bash title:"Например, в snmp.conf:" unfold
extend .1.3.6.1.4.1.9999.1.1 myString echo "Hello SNMP"
extend .1.3.6.1.4.1.9999.1.2 myCounter echo 123
```
Теперь агент отдаёт твои объекты.
2. Написать свой SNMP-агент (C++)
Использовать **Net-SNMP Agent++** (C++ API).
Или писать с нуля (UDP/161, ASN.1 кодирование), но это долго.

```cpp title="Тестовая программа менеджера на C++"
#include <net-snmp/net-snmp-config.h>
#include <net-snmp/net-snmp-includes.h>
#include <iostream>

int main() {
    // Инициализация
    init_snmp("snmpdemoapp");
    SOCK_STARTUP;

    // Настраиваем сессию
    snmp_session session, *ss;
    snmp_sess_init(&session);                // Инициализируем структуру
    session.peername = strdup("127.0.0.1");  // Агент (localhost)
    session.version = SNMP_VERSION_2c;
    session.community = (u_char*)"public";   // community string
    session.community_len = strlen((char*)session.community);

    ss = snmp_open(&session);
    if (!ss) {
        snmp_perror("snmp_open");
        return 1;
    }

    // Создаем PDU (Get запрос)
    netsnmp_pdu *pdu = snmp_pdu_create(SNMP_MSG_GET);
    netsnmp_pdu *response;

    // Добавляем несколько OID (например sysName и sysUpTime)
    oid anOID[MAX_OID_LEN];
    size_t anOID_len;

    anOID_len = MAX_OID_LEN;
    if (!snmp_parse_oid("MY-TEST-MIB::myString.0", anOID, &anOID_len)) {
        snmp_perror("sysName");
        return 1;
    }
    snmp_add_null_var(pdu, anOID, anOID_len);

    anOID_len = MAX_OID_LEN;
    if (!snmp_parse_oid("MY-TEST-MIB::myCounter.0", anOID, &anOID_len)) {
        snmp_perror("sysUpTime");
        return 1;
    }
    snmp_add_null_var(pdu, anOID, anOID_len);

    // Отправляем запрос
    int status = snmp_synch_response(ss, pdu, &response);

    if (status == STAT_SUCCESS && response->errstat == SNMP_ERR_NOERROR) {
        // Обрабатываем результат
        for (netsnmp_variable_list *vars = response->variables; vars; vars = vars->next_variable) {
            char buf[1024];
            snprint_variable(buf, sizeof(buf), vars->name, vars->name_length, vars);
            std::cout << buf << std::endl;
        }
    } else {
        std::cerr << "SNMP request failed." << std::endl;
    }

    // Очистка
    if (response) snmp_free_pdu(response);
    snmp_close(ss);
    SOCK_CLEANUP;
    return 0;
}
```
При опросе SNMP-агента получаешь:
```text unfold
MY-TEST-MIB::myString.0 = STRING: Hello SNMP
MY-TEST-MIB::myCounter.0 = INTEGER: 123
```