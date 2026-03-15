
---
### Теги

### [[Информация|Назад]]
### Далее
####
---

### Типы
| Тип интерфейса | Для чего нужен                                                                                                                                                                       | Уровень    |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| TAP            | Передает целые Ethernet-кадры (с MAC-адресами). Позволяет **пользовательским программам** (вроде эмуляторов) напрямую читать и писать в сетевой стек ядра                            |            |
|                |                                                                                                                                                                                      |            |
| **TUN**        | То же что TAP, но работает на **третьем уровне (L3, сетевой)**. Передает IP-пакеты (без MAC-адресов). Используется в OpenVPN [](https://cloud.tencent.cn/developer/article/2133967). | L3         |
| **Bridge**     | Виртуальный **сетевой коммутатор**. Соединяет несколько интерфейсов в один L2-сегмент [](https://ostinato.org/guides/linux-virtual-network-interfaces).                              | L2         |
| **VLAN**       | Для нарезания одного физического интерфейса на несколько логических сетей с тегированием трафика (802.1q) [](https://ostinato.org/guides/linux-virtual-network-interfaces).          | L2         |
| **MACVLAN**    | Позволяет создать несколько виртуальных интерфейсов с **разными MAC-адресами** на одной физической карте [](https://ostinato.org/guides/linux-virtual-network-interfaces).           | L2         |
| **IPVLAN**     | То же, что MACVLAN, но все виртуальные интерфейсы используют **один MAC**, а идентификация идет по IP [](https://ostinato.org/guides/linux-virtual-network-interfaces).              | L2/L3      |
| **MACVTAP**    | Упрощенная замена комбинации TAP + MACVLAN для виртуализации [](https://ostinato.org/guides/linux-virtual-network-interfaces).                                                       | L2         |
| **VXLAN**      | Для создания оверлейных сетей поверх UDP (используется в Kubernetes, облаках) [](https://ostinato.org/guides/linux-virtual-network-interfaces).                                      | L2 over L3 |
| **GRE/IPIP**   | Туннели для упаковки одного сетевого протокола в другой [](https://ostinato.org/guides/linux-virtual-network-interfaces).                                                            | L3         |
| **MACsec**     | Для шифрования трафика на канальном уровне (802.1AE) [](https://ostinato.org/guides/linux-virtual-network-interfaces).                                                               | L2         |
| **Bonding**    | Объединение нескольких физических интерфейсов в один для отказоустойчивости или увеличения пропускной способности [](https://ostinato.org/guides/linux-virtual-network-interfaces).  | L2         |
| **Dummy**      | "Заглушка". Всегда поднят, нужен для тестов или чтобы программа думала, что сеть есть [](https://ostinato.org/guides/linux-virtual-network-interfaces).                              | L3         |
