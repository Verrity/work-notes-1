
---
### Теги
#esr_rep 
#esr_clish 
### Назад
#### [[ESR]]
### Далее
####
---

#### Использование переменных в XML для конкретных типов
В`clish/xml-files/common/namespace-do.xml`есть переменные. Как я понял, они обязательны к использованию при каких-то типах.
```xml folded
		<COMMAND name="nas-ip-address"
		         help="Set NAS ip address for proxy mode"
		         permissions="10">
			<PARAM name="IP_SOURCE_ADDR4"
			       ptype="TYPE_IP_ADDR"
			       names-request="CA|ip-interfaces|ifaces|#|OBJ_GET_ELEM_NAMES|ipv4"
			       help="IP address of NAS"/>
			<ACTION builtin="klish_reflection_handler">"${_IPC_PATH_BASE}|#|ATTR_SET|nas-ip-address|${IP_SOURCE}"</ACTION>
		</COMMAND>
		<COMMAND name="nas-ip-address object-group"
		         permissions="10"
		         help="Object group">
			<PARAM name="IP_SOURCE_OBJ_GROUP"
			       ptype="TYPE_RESTRICTED_NAME_NO_ANY"
			       names-request="CA|object-group|networks|#|OBJ_GET_ELEM_NAMES"
			       help="Name of network object group. Name 'any' reserved"/>
			<ACTION builtin="klish_reflection_handler">"${_IPC_PATH_BASE}|#|ATTR_SET|nas-ip-address|${IP_SOURCE}"</ACTION>
		</COMMAND>
```

  

Если использовать тип IP_SOURCE (это типо union на`ip-address`,`object-group`,`interface-name`, `id`, см.  `ESRconfig_ip_source_type_t`), то нужно:
В`<PARAM>`-->`name`указывать определенное имя, как`IP_SOURCE_ADDR4`,`IP_SOURCE_OBJ_GROUP`.
И в`<ACTION>`вместо имени параметра передавать тоже специальную переменную`${IP_SOURCE}`.
