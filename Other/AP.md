
---
### Теги

### [[Home|Назад]]
#### 
### Далее
####
---

### Добавление фейковых клиентов
```bash unfold
wlcctl -r 'DISABLE_AP_MONITORING|68:13:e2:1f:59:80|true'
```
```bash unfold
wlcctl -r 'ADD_FAKE_CLIENTS|68:13:e2:1f:59:80|3|82:00:13:70:20:00'
```
```bash unfold
wlcctl -r 'GET_CLIENT_MACLIST|{"client-mac":"8"}|{"limit":"128"}'
```
### Включить логи в консоли
(server - в нем сертификаты)
```unfold
configure
service-activator
wlc
debug true
logfile /dev/console
server
debug true
logfile /dev/console
save
exit
exit
exit
exit
```


 Таблица проектов и ТД из `board-types.json`

|                                     |            |                 |             |                 |                 |
| ----------------------------------- | ---------- | --------------- | ----------- | --------------- | --------------- |
| **realtek-ap-enterprise-2**         | `WEP-1L`   | `WEP-2L`        | `WOP-2L`    |                 |                 |
| **realtek-ap-enterprise**           | `WEP-20L`  | `WOP-20L`       | `WEP-200L`  |                 |                 |
| **realtek-ap-ax-enterprise**        | `WEP-30L`  | `WOP-30L`       | `WEP-30L-Z` | `WOP-30LI`      | `WEP-30L-NB`    |
| **realtek-ap-bgn-ax-80-enterprise** | `WEP-3L`   | `WOP-3L-EX`     |             |                 |                 |
| **realtek-ap-ax-s-enterprise**      |            | `WOP-30LS`      |             |                 |                 |
| **broadcom-ap-enterprise**          | `WEP-3ax`  | `WOP-3ax`       | `WEP-3ax-Z` |                 |                 |
| **mediatek-ap-enterprise**          | `WEP-550K` |                 |             |                 |                 |
| **fastpath-ap-enterprise**          | `WEP-2ac`  | `WEP-2ac Smart` | `WOP-2ac`   | `WOP-2ac:rev.B` | `WOP-2ac:rev.C` |

 Таблица проектов и ТД из `Redmine`

|                               |               |                 |                   |            |
| ----------------------------- | ------------- | --------------- | ----------------- | ---------- |
| **ESDK**                      | `WEP-3ax`     |                 |                   |            |
| **Fastpath**                  | `WEP-12ac`    | `WEP-2ac`       |                   |            |
| **MediaTek**                  | `WEP-500K`    | `WEP-550K`      | `WEP-550K(S)`     |            |
| **Realtek**                   | `WEP-1L`      | `WEP-2L`        | `WOP-2L`          |            |
|                               | `WEP-200L`    | `WEP-20L`       |                   |            |
|                               | `WEP-30L`     | `WOP-30L`       | `WOP-30LI`        | `WOP-30LS` |
|                               | `WEP-3L`      | `WOP-3L-EX`     |                   |            |
|                               | `WEP-50L`     | `WEP-53L`       | `WOP-50L`         |            |
| **(Realtek-FBWA) Realtek-ac** | `WB-2P-LR2`   | `WB-2P-LR5`     | `WB-2P-LRx-WEB`   |            |
|                               | `WOP-2ac-LR2` | `WOP-2ac-LR5`   | `WOP-2ac-LRx-WEB` |            |
| **(Realtek-FBWA) Realtek-ax** | `WB-3P-LR2`   | `WB-3P-LR5/6`   |                   |            |
|                               | `WB-3P-PTP2`  | `WB-3P-PTP5/6`  |                   |            |
|                               | `WOP-3ax-LR2` | `WOP-3ax-LR5/6` |                   |            |


