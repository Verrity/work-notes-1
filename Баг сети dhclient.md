
---
### Теги

### [[System|Назад]]
### Далее
####
---

# Разбор проблемы: «Кто-то удаляет мой 192.168.1.111 и ставит 192.168.1.1»

## Оглавление

- [Краткое резюме](#краткое-резюме)
- [1. Как возникла проблема](#1-как-возникла-проблема)
- [2. Как проявлялась проблема](#2-как-проявлялась-проблема)
- [3. Как это работало технически](#3-как-это-работало-технически)
- [4. Как исправляется](#4-как-исправляется)
- [5. Как предотвратить появление](#5-как-предотвратить-появление)
- [Приложение A: Диагностические команды](#приложение-a-диагностические-команды)
- [Приложение B: Хронология расследования](#приложение-b-хронология-расследования)

---

## Краткое резюме

**Проблема:** статический адрес `192.168.1.111` на `enp1s0` периодически заменялся на динамический `192.168.1.10`, из-за чего пропадал интернет.

**Причина:** в терминале были вручную запущены два процесса `dhclient` **без указания интерфейса** (PID 24663 и 153381). Они демонизировались, попали в `vte-spawn-*.scope` (скоуп терминала, **не** systemd-сервис), обходили `dhcp4: false` из netplan и каждые ~9 минут продлевали аренду, перезаписывая `.111` на `.10`.

**Исправление:** `sudo pkill -x dhclient` + `sudo netplan apply`.

**Профилактика:** не запускать голый `sudo dhclient`; добавлять `pkill -x dhclient` в скрипт; при подозрении — сразу проверять `ss -ulpn 'sport = :68'`.

---

## 1. Как возникла проблема

### 1.1. Первопричина

В какой-то момент в терминале `gnome-terminal` (внутри сессии `zellij`) была выполнена команда:

```bash
sudo dhclient
```

**Без аргументов-интерфейсов.** Это критический момент. По документации ISC:

> «Если имя интерфейса не указано, `dhclient` работает со **всеми** сетевыми интерфейсами, для которых есть конфигурация.»

То есть `dhclient` **обходит все интерфейсы**, включая `enp1s0`, у которого в netplan стоит `dhcp4: false`. Netplan тут ни при чём — `dhclient` читает только свой конфиг и **игнорирует** настройки netplan.

Аналогичная команда была выполнена для `enp2s0`:

```bash
sudo dhclient enp2s0
```

### 1.2. Почему процессы «забылись»

Оба процесса **демонизировались** — форкнулись в фон, отцепились от терминала и продолжили работать. В результате они оказались в:

```
vte-spawn-242f3bc7-dff1-4113-93f9-47bfe1055685.scope
```

Это **скоуп** — контейнер cgroup, созданный `gnome-terminal-server` для процессов терминальной вкладки. Скоуп:

- **не является** `systemd`-сервисом;
- **не имеет** `Restart=`, `enable`, автозапуска;
- **не виден** через `systemctl status dhclient` или `systemctl list-units --all 'dhclient*'`;
- **не виден** через `systemctl --user list-units | grep dhclient`.

Именно поэтому ни скрипт пользователя, ни стандартные команды управления службами этих «сирот» не находили.

### 1.3. Подтверждение из вывода расследования

```
$ systemctl --user status 24663
● vte-spawn-242f3bc7-....scope - VTE child process 7005 launched by gnome-terminal-server
  Transient: yes
  CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/app-org.gnome.Terminal.slice/vte-spawn-242f3bc7-...scope
          ├─ 24663 dhclient
          └─ 153381 dhclient enp2s0

$ systemctl --user list-units --all | grep -i dhclient
(пусто)
```

**Вывод:** оба `dhclient` — это «сироты», запущенные руками. Никакого сервиса за ними нет.

---

## 2. Как проявлялась проблема

### 2.1. Пользовательские симптомы

- **«Кто-то удаляет мой 192.168.1.111 и ставит 192.168.1.1»** — фактически вместо `.111` вставал `.10`, но суть та же.
- **Интернет пропадал** — потому что `enp1s0` начинал светиться в чужой подсети, и маршруты/соединения рвались.
- **`--restore` временно возвращал `.111`**, интернет появлялся, потом снова отваливался.
- **Перезагрузка/перенастройка сети помогала на время.**

### 2.2. Симптомы в системе

Команда `ip -4 addr show enp1s0`:

```
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.10/24 brd 192.168.1.255 scope global dynamic enp1s0
       valid_lft 461sec preferred_lft 461sec
```

- Адрес `.10` вместо `.111`.
- Ключевое слово **`dynamic`** — признак того, что адрес получен по DHCP, а не задан статически.
- `valid_lft` — признак аренды с таймаутом.

### 2.3. Ложные следы в логах

В `journalctl -u systemd-networkd` по `enp1s0`:

```
enp1s0: DHCPv6 lease lost
enp1s0: DHCPv6 lease lost
...
```

**Ни одной строки `DHCPv4 address ... acquired`.** Это была первая зацепка: networkd **не получал IPv4 по DHCP**, значит, его получал кто-то другой.

В `journalctl -u isc-dhcp-server`:

```
DHCPOFFER on 192.168.1.5  to e8:28:c1:32:01:40
DHCPOFFER on 192.168.1.7  to 90:54:b7:c1:1f:30 (WEP-30L)
DHCPOFFER on 192.168.1.10 to 8c:90:2d:fd:ab:9d (daniil.levin.eltex.loc)
```

**Ни одного `DHCPOFFER` на `.111`.** То есть первоначальная гипотеза «DHCP-сервер раздаёт `.111`» — **не подтвердилась**. Пул `.3–.111` содержит `.111`, но фактической выдачи нет.

В `journalctl -u systemd-networkd` по `enp2s0`:

```
enp2s0: Lost carrier
enp2s0: Gained carrier
enp2s0: DHCPv4 address 10.30.111.92/24, gateway 10.30.111.1 acquired from 10.30.0.10
```

Сотни `Lost carrier` — но это **побочный эффект** перезапусков `netplan apply` и `ip link set down` из скрипта, а **не** физическая проблема кабеля. Пользователь был прав, когда это утверждал.

### 2.4. Прямое доказательство через `tcpdump`

Перехват на `enp1s0` портов 67/68 показал полный DHCP-цикл:

```
11:27:02.961721 IP 0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request
          Client-Ethernet-Address 8c:90:2d:fd:ab:9d
          Hostname: "daniil.levin.eltex.loc"
          DHCP-Message: Discover
          Requested-IP: 192.168.1.10

11:27:02.961798 IP 192.168.1.111.67 > 192.168.1.10.68: BOOTP/DHCP, Reply
          Your-IP 192.168.1.10
          Server-IP 192.168.1.111
          DHCP-Message: Offer
          Default-Gateway: 192.168.1.111
          Lease-Time: 544

11:27:02.961855 IP 192.168.1.111.67 > 192.168.1.10.68: BOOTP/DHCP, Reply
          Your-IP 192.168.1.10
          Server-IP 192.168.1.111
          DHCP-Message: ACK
          Default-Gateway: 192.168.1.111
```

**Это прямое доказательство вины `dhclient`:** машина сама себе (MAC `enp1s0`) запросила `.10` у своего же `isc-dhcp-server` на `192.168.1.111` и получила ACK.

---

## 3. Как это работало технически

### 3.1. Пошаговый механизм

1. **`dhclient` (без аргументов)** находит `enp1s0` и запускает на нём DHCP-клиент, **игнорируя `dhcp4: false` из netplan** — netplan тут ни при чём, `dhclient` читает только свой конфиг (`/etc/dhcp/dhclient.conf`).

2. **`dhclient` шлёт `DHCPDISCOVER`** с MAC `8c:90:2d:fd:ab:9d` (MAC `enp1s0`) и hostname `daniil.levin.eltex.loc`.

3. **Собственный `isc-dhcp-server`** на `192.168.1.111` отвечает:
   - `OFFER` с `Your-IP 192.168.1.10`, `Server-IP 192.168.1.111`, `Default-Gateway 192.168.1.111`;
   - `ACK` с тем же `192.168.1.10`, `Lease-Time 544`.

   Почему `.10`? Потому что `.10` входит в `range 192.168.1.3 192.168.1.111` и был свободен (старая аренда по MAC `8c:90:2d:fd:ab:9d`).

4. **`dhclient-script`** (стандартный хук Debian/Ubuntu) при получении `ACK`:
   - делает **`ip addr flush dev enp1s0`** — снимает **все** адреса, включая статический `.111`;
   - ставит `.10/24 dynamic`.
   
   Именно поэтому `.111` **исчезал полностью**, а не висел рядом вторым адресом.

5. **`dhclient` продолжает работать** и каждые ~544 секунды (по `default-lease-time` / `max-lease-time` из `dhcpd.conf`) **продлевает аренду** — снова и снова ставя `.10` поверх `.111`.

6. **Когда пользователь запускал `--restore`:**

   ```
   netplan_stop              → стоп systemd-networkd
   network_clspa_cleanup     → ip addr flush dev enp1s0, снос bridge/vlan/tap
   netplan_restore_default   → симлинк на 01-networkd-default.yaml + netplan apply
   netplan_start             → networkd запущен
   ```

   Networkd ставит `.111`. **Но `dhclient` жив** — его никто не убивал. Как только линк на `enp1s0` мигнул (`netplan apply` дёргает `ip link set down/up`), `dhclient` либо обновляет аренду, либо делает новый `DISCOVER` → снова `flush` → снова `.10`.

7. **Цикл замыкается:**

   ```
   интернет есть → через минуту пропал → сделал --restore → снова есть → через минуту пропал → ...
   ```

### 3.2. Схема цикла

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Пользователь: ./script --restore                           │
│         │                                                   │
│         ▼                                                   │
│  networkd ставит .111/24 (static)                           │
│         │                                                   │
│         ▼                                                   │
│  Интернет работает                                         │
│         │                                                   │
│         ▼                                                   │
│  dhclient (orphan) ждёт renew (≤544s)                       │
│         │                                                   │
│         ▼                                                   │
│  dhclient → DISCOVER → isc-dhcp-server → OFFER/ACK .10      │
│         │                                                   │
│         ▼                                                   │
│  dhclient-script: ip addr flush dev enp1s0                  │
│                   ip addr add .10/24 dynamic                │
│         │                                                   │
│         ▼                                                   │
│  .111 ИСЧЕЗ, маршруты сломаны, интернет пропал              │
│         │                                                   │
│         └──────────────────► (goto start)                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.3. Почему `Lost carrier` на `enp2s0` — это не физика

`ip link set down` в `network_clspa_cleanup` и `netplan apply` в `netplan_restore_default` **дёргают интерфейс** `enp2s0`. Драйвер пишет `Lost carrier` / `Gained carrier`, потом `DHCPv4 ... acquired` от провайдера. В логах выглядело как «пропадает линк», но на самом деле это **собственный скрипт** его дёргал.

Проверка: если `Lost carrier` идёт **без** предшествующего `Configuring with /run/systemd/network/...` или `ip link set ... down` — это физика. Если с предшествующим — это скрипт.

### 3.4. Почему `192.168.24.16` на `enp2s0` — тоже не физика

В логе был эпизод:

```
Sep 21 08:58:13 enp2s0: DHCPv4 address 192.168.24.16/24, gateway 192.168.24.1 acquired from 192.168.24.1
Sep 21 09:58:13 enp2s0: DHCPv4 address 10.30.111.92/24, gateway 10.30.111.1 acquired from 10.30.0.10
```

Целый час `enp2s0` был в сети `192.168.24.0/24`. Это либо кабель перетыкали, либо роутер переключался между двумя сетями. Но это **разовое событие**, не связанное с основной проблемой.

---

## 4. Как исправляется

### 4.1. Единственная команда

```bash
sudo pkill -x dhclient
```

`-x` — точное совпадение имени (`dhclient`, а не `dhclient6` или `mydhclient`). После этого оба «сироты» (PID 24663 и 153381) умирают **навсегда**, потому что:

- их не держит `systemd`-сервис (`systemctl --user list-units | grep dhclient` пуст);
- их не перезапускает `scope` (`vte-spawn-*` не имеет `Restart=`);
- их не поднимет `systemd-networkd` (он использует свой встроенный DHCP-клиент, не `dhclient`).

### 4.2. Финальный шаг — пнуть networkd

После `pkill` адрес `.10` **остаётся в ядре** — networkd не переустанавливает конфиг сам по себе, он реагирует только на события. Нужно принудительно:

```bash
sudo netplan apply
```

Если `.10` упорно висит:

```bash
sudo ip addr del 192.168.1.10/24 dev enp1s0
sudo netplan apply
```

### 4.3. Проверка

```bash
ip -4 addr show enp1s0
```

Должно быть:

```
inet 192.168.1.111/24 brd 192.168.1.255 scope global enp1s0
```

**Без `dynamic`** и **без `valid_lft`** — значит, адрес снова статический из netplan.

```bash
ps -ef | grep -w dhclient | grep -v grep
```

Должно быть пусто.

### 4.4. Почему это окончательное решение

Поскольку `dhclient` больше нет, а юнита, который бы его поднимал, в системе тоже нет (`systemctl --user list-units | grep dhclient` пуст, `systemctl list-units --all 'dhclient*'` пуст), **никто не сможет перезаписать `.111`**. Проблема закрыта навсегда, если не запустить `dhclient` снова вручную.

---

## 5. Как предотвратить появление

### Правило №1: никогда не запускай голый `dhclient`

```bash
# ❌ НИКОГДА — обойдёт все интерфейсы, включая dhcp4: false
sudo dhclient

# ✅ Если нужно обновить аренду на конкретном интерфейсе — один раз и выйти
sudo dhclient -1 -v enp2s0

# ✅ Если нужно освободить аренду
sudo dhclient -r enp2s0
```

- `-1` = один раз и завершиться, **без демонизации**.
- `-v` = подробный вывод, чтобы видеть, что происходит.
- **Всегда указывай интерфейс явно**.

### Правило №2: добавь защиту в скрипт

В `netplan_stop()` добавь гарантированную зачистку «сирот»:

```bash
netplan_stop() {
    print_info "Остановка systemd-networkd и его сокета..."
    sudo systemctl stop systemd-networkd 2>/dev/null || true
    sudo systemctl stop systemd-networkd.socket 2>/dev/null || true

    print_info "Убийство забытых dhclient, перезаписывающих статические IP..."
    sudo pkill -x dhclient 2>/dev/null || true
}
```

В `network_clspa_cleanup()` уже есть `ip addr flush dev "$ETH"` — это правильно, оставь как есть.

### Правило №3: знай, как найти виновника за 5 секунд

Если снова начнёт «уплывать» `.111` — **первая команда**:

```bash
sudo ss -ulpn 'sport = :68'
```

Она покажет **PID и имя** процесса, держащего DHCP-клиентский порт 68. Пример вывода:

```
users:(("dhclient",pid=24663,fd=13))
users:(("dhclient",pid=153381,fd=7))
users:(("systemd-network",pid=150967,fd=24))   # ← это нормально, networkd для enp2s0
```

Интерпретация:

| Что видно | Что делать |
|---|---|
| `dhclient` | `sudo pkill -x dhclient` |
| `systemd-network` | Это netplan/networkd, копать в `/etc/netplan/` |
| `dhcpcd`, `NetworkManager` | Разбираться с тем менеджером |
| `dhclient` в `@enp1s0.service` | `systemctl disable --now dhclient@enp1s0` |

### Правило №4: понимай разницу между scope и service

| | `vte-spawn-*.scope` | `*.service` |
|---|---|---|
| Что это | Контейнер cgroup для процессов терминала | Юнит systemd |
| Перезапуск | **Нет** | Может быть (`Restart=always`) |
| Автозапуск | **Нет** | Может быть (`enable`) |
| Убил процесс | Умер навсегда | Может перезапуститься |
| Как остановить | `kill` / `pkill` | `systemctl disable --now <unit>` |

В нашем случае был **scope**, а не service. Поэтому `systemctl disable` ничего бы не нашёл, и `pkill` — **финальное** решение, а не временное.

### Правило №5: не путай «пропал интернет» с «пропал линк»

`Lost carrier` в `journalctl` **не всегда** означает физическую проблему. Если перед ним видишь:

- `Configuring with /run/systemd/network/...`
- `ip link set ... down` в истории команд
- или это совпадает с запуском `--restore`/`--setup`

— это **твой же скрипт** дёргает интерфейс, а не кабель.

Физику проверяй только если `Lost carrier` идёт **без** предшествующей реконфигурации.

---

## Приложение A: Диагностические команды

### A.1. Кто держит DHCP-клиент

```bash
sudo ss -ulpn 'sport = :68'
```

### A.2. Кто управляет интерфейсом (все менеджеры)

```bash
ps aux | grep -E "dhclient|NetworkManager|ifup|wpa_supplicant|systemd-network"
systemctl is-active NetworkManager dhclient networking systemd-networkd
```

### A.3. Что думает networkd

```bash
networkctl status enp1s0
networkctl status enp2s0
```

### A.4. Кто в `/etc/network/interfaces`

```bash
cat /etc/network/interfaces 2>/dev/null
cat /etc/network/interfaces.d/* 2>/dev/null
```

### A.5. Что в netplan

```bash
ls -la /etc/netplan/
sudo netplan get
```

### A.6. Поймать DHCP в эфире

```bash
sudo tcpdump -i enp1s0 -n -v port 67 or port 68 -w /tmp/dhcp.pcap
# оставить на 10-15 минут
sudo tcpdump -r /tmp/dhcp.pcap -n -v
```

### A.7. Смотреть изменения в реальном времени

```bash
# В одном окне:
watch -n 1 'ip -4 addr show enp1s0; echo ---; ip route; echo ---; systemctl is-active NetworkManager dhclient networking systemd-networkd'

# В другом:
sudo journalctl -f | grep -iE "enp1s0|dhclient|NetworkManager|systemd-networkd"
```

### A.8. Найти «сирот» в scope

```bash
systemctl --user list-units --all --type=scope
systemctl --user status <PID>
cat /proc/<PID>/cgroup
pstree -sp <PID>
```

### A.9. Найти юнит по процессу

```bash
systemctl status <PID>        # системный
systemctl --user status <PID>  # пользовательский
```

---

## Приложение B: Хронология расследования

| # | Гипотеза | Как проверялась | Результат |
|---|---|---|---|
| 1 | DHCP-сервер раздаёт `.111` из `range` | `journalctl -u isc-dhcp-server \| grep DHCPOFFER` | ❌ Не подтвердилось: только `.5`, `.7`, `.10` |
| 2 | `systemd-networkd` перезаписывает маршруты на `enp2s0` | `journalctl -u systemd-networkd \| grep enp2s0` | ❌ Не подтвердилось как причина, но подтвердилось как поведение (это side-effect) |
| 3 | `systemd-networkd` перезаписывает адрес на `enp1s0` | `journalctl -u systemd-networkd \| grep enp1s0` | ❌ Не подтвердилось: нет `DHCPv4 acquired` |
| 4 | Физика на `enp2s0` (`Lost carrier`) | Корреляция с `--restore` | ❌ Не подтвердилось: side-effect `netplan apply` |
| 5 | Второй DHCP-сервер в сети `enp1s0` | `tcpdump` портов 67/68 | ❌ Не подтвердилось: только наш `.111` |
| 6 | Кто-то ставит `.10` через DHCP | `tcpdump` — полный цикл DISCOVER/OFFER/ACK | ✅ **Подтверждено**: MAC `enp1s0` сам просит `.10` |
| 7 | Кто именно шлёт DHCP | `ss -ulpn 'sport = :68'` | ✅ **Подтверждено**: `dhclient` PID 24663 и 153381 |
| 8 | Кто запустил `dhclient` | `pstree -sp 24663`, `systemctl --user status 24663` | ✅ **Подтверждено**: orphan-процессы в `vte-spawn-*.scope`, запущены вручную |
| 9 | Есть ли юнит, который их перезапустит | `systemctl --user list-units \| grep dhclient` | ✅ **Подтверждено**: юнитов нет, `pkill` — окончательное решение |

---

## Итоговая формула

```
Проблема:      .111 → .10 → интернет пропал
Причина:       два orphan-dhclient в vte-spawn-*.scope
Механизм:      dhclient-script делает ip addr flush перед установкой .10
Исправление:   sudo pkill -x dhclient && sudo netplan apply
Профилактика:  никогда не запускать голый sudo dhclient
               + добавить pkill в netplan_stop()
               + при симптомах сразу ss -ulpn 'sport = :68'
```
