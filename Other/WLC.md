
---
### Теги
#wlc_rep
### [[Home|Назад]]
### Далее
#### [[Ipc]]
---

`WLC` может запускаться в `stand alone` режими на `esr`, когда `esr` работает без лицензии `wlc`, на `wlc` запускается `sevice validator` или как-то так, когда активен только он. Другие компоненты не поднимаются. При этом можно использовать некотоые функции управления журналом.

#### Создать фейковых клиентов
```bash fold title="wlcctl help"
wlcctl -r 'DISABLE_AP_MONITORING|68:13:e2:c2:d1:c0|true'
wlcctl -r 'ADD_FAKE_CLIENTS|68:13:e2:c2:d1:c0|1000|82:00:13:70:20:00'
wlcctl -r 'GET_CLIENT_MACLIST|{"client-mac":"8"}|{"limit":"128"}'
```