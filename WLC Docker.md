
---
### Теги

### [[WLC|Назад]]
### Далее
####
---

Требуется:
* DHCP
	* Включить сервис `isc-dhcp-server`
	* Проверить что в `/etc/default/isc-dhcp-server` в переменная `INTERFACESv4="enp1s0"` установлена на интерфейс, уходящий в вашу сеть
	* Проверить что в `/etc/dhcp/dhcpd.conf` закодирован адрес ПК (особенно, что длина после `OF` корректная)
		```conf unfold
		subnet 192.168.1.0 netmask 255.255.255.0 {
			option routers 192.168.1.111;
			range 192.168.1.3 192.168.1.111;
			# 0F:<data_len>:<data...>;
			# OF:1C (28): https://192.168.1.111:8043
			option vendor-encapsulated-options 0F:1A:68:74:74:70:73:3A:2F:2F:31:39:32:2E:31:36:38:2E:31:2E:31:31:31:3A:38:30:34:33;
		}
		```
* db.json
	* Проверить что в качестве `common/base-address` стоит адрес вашего ПК
	* Проверить что в `service-activator/auto-authorize-aps` стоит `true`
	* Проверить что `service-activator/base-port` стоит `8043`
	* Проверить что пути сертификатов `SA` являются докерными:
		```json unfold
		"certificate": {
			"private-key-password": "",
			"ca-key": "sa.key",
			"ca-cert": "sa.cert"
		}
		```