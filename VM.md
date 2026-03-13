
---
### Теги

### [[System|Назад]]
### Далее
####
---

# Установка

### Шаг 1. Установка необходимых пакетов
Обновите список пакетов и установите QEMU, инструменты для управления мостами и виртуализацией
```bash unfold
sudo apt update
sudo apt install bridge-utils qemu-kvm libvirt-daemon-system libvirt-clients virtinst qemu-system-x86
```

### Шаг 2. Включение IP-форвардинга
Для работы сетевых мостов и маршрутизации между ними необходимо разрешить пересылку пакетов между интерфейсами
```conf unfold title="/etc/sysctl.conf"
net.ipv4.ip_forward=1
```
```bash unfold
sudo sysctl -p
```

### Шаг 3. Настройка сетевых мостов через netplan
Создайте или отредактируйте файл конфигурации netplan В примере ниже используются интерфейсы enp1s0 (локальная сеть) и enp2s0 (интернет). Подставьте свои имена интерфейсов и нужные IP-адреса
```yaml fold title="/etc/netplan/something.yaml"
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:          # Local network
      dhcp4: false
      dhcp6: false
    enp2s0:          # Internet network
      dhcp4: false
      dhcp6: false
  bridges:
    br-local:        # Local network bridge
      addresses:
        - "192.168.1.111/24"
      interfaces:
        - enp1s0
    br-wan:          # Internet network bridge
      dhcp4: true
      dhcp6: true
      interfaces:
        - enp2s0
```
```bash unfold
sudo netplan apply
```

### Шаг 4. Настройка QEMU для использования мостов
QEMU использует вспомогательную программу qemu-bridge-helper для подключения виртуальных машин к мостам. Нужно разрешить использование мостов в конфигурации

Создайте каталог для конфигурации QEMU (если его нет):
```bash unfold
sudo mkdir -p /etc/qemu
```
```conf file title="/etc/qemu/bridge.conf"
allow br-local
allow br-wan
```
Установите правильные права:
```bash unfold
sudo chmod 644 /etc/qemu/bridge.conf
```

Установите SUID-бит на `qemu-bridge-helper`, чтобы QEMU мог использовать его даже от обычного пользователя (необходимо для запуска ВМ без sudo):
```bash unfold
sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

Примечание: Путь к qemu-bridge-helper может отличаться в зависимости от версии. Проверьте его командой 
```bash fold
find /usr -name qemu-bridge-helper 2>/dev/null
```

### Шаг 5. Добавление пользователя в группу kvm
Чтобы иметь доступ к устройствам KVM, добавьте своего пользователя в группу kvm:
```bash unfold
sudo usermod -aG kvm $USER
```

### Шаг 6. Создание образа диска для виртуальной машины
Создайте каталог для дисков и образов (по желанию) и сформируйте qcow2-образ размером 50 ГБ:
```bash unfold
mkdir -p ~/vm/disks
qemu-img create -f qcow2 ~/vm/disks/ubuntu-disk.qcow2 50G
```

#### Шаг 7. Установка гостевой ОС
Для установки Ubuntu Server потребуется ISO-образ. Скачайте его, например, в каталог `~/vm/images/`. Затем выполните команду запуска QEMU с параметрами:
```bash fold
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 4096 \
  -smp 2 \
  -drive file=/home/daniil/vm/disks/ubuntu-disk.qcow2,format=qcow2 \
  -cdrom /home/daniil/vm/images/ubuntu-22.04.5-live-server-amd64.iso \
  -netdev bridge,id=net0,br=br-local \
  -device virtio-net-pci,netdev=net0,mac=52:54:00:12:34:01 \
  -netdev bridge,id=net1,br=br-wan \
  -device virtio-net-pci,netdev=net1,mac=52:54:00:12:34:02 \
  -display gtk \
  -vga virtio
```

Пояснения параметров:
* `enable-kvm` – использовать аппаратную виртуализацию.
* `cpu host` – передать гостю все возможности хост-процессора.
* `m 4096` – 4 ГБ ОЗУ. Указывается в мегабайтах. Например, для выделения 16 ГБ используйте -m 16384, для 32 ГБ — -m 32768.
* `smp 2` – Количество ядер. Задаётся числом, например, -smp 8 для 8 виртуальных CPU.
* `drive` – подключает созданный образ как диск.
* `cdrom` – подключает ISO для установки.
* `netdev bridge` – создаёт сетевые интерфейсы, привязанные к мостам br-local и br-wan.
* `device virtio-net-pci` – задаёт тип сетевой карты и её MAC-адреса.
* `display gtk` – открывает графическое окно (можно заменить на -nographic для консоли).
* `vga virtio` – тип видеокарты.

*Убедитесь, что на вашем физическом сервере достаточно свободной оперативной памяти (команда `free -h`) и свободных ядер CPU (можно посмотреть загрузку через `htop` или `lscpu`).*

### Шаг 8. Настройка сети внутри гостевой системы
После запуска установщика следуйте инструкциям. На этапе настройки сети вам будут доступны два интерфейса:
* Первый интерфейс (подключён к br-local, локальная сеть). Настройте его вручную:
	* IP-адрес: 192.168.1.200 (или любой свободный в вашей локальной подсети)
	* Маска: 255.255.255.0
	* Шлюз: оставьте пустым (если не требуется маршрутизация в локальную сеть через хост; обычно не нужен).
* Второй интерфейс (подключён к br-wan, интернет). Настройте его на получение адреса по DHCP (автоматически). Тогда виртуальная машина получит IP от роутера и доступ в интернет.

**Важно**: MAC-адреса, указанные при запуске, должны быть уникальны в вашей локальной сети. При необходимости измените их.
После завершения установки виртуальная машина перезагрузится. Закройте окно QEMU.

### Шаг 9. Запуск установленной виртуальной машины
Для последующего запуска (без установочного ISO) используйте ту же команду, но без параметра -cdrom:
```bash fold title="Команда запуска"
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 4096 \
  -smp 2 \
  -drive file=/home/daniil/vm/disks/ubuntu-disk.qcow2,format=qcow2 \
  -netdev bridge,id=net0,br=br-local \
  -device virtio-net-pci,netdev=net0,mac=52:54:00:12:34:01 \
  -netdev bridge,id=net1,br=br-wan \
  -device virtio-net-pci,netdev=net1,mac=52:54:00:12:34:02 \
  -display gtk \
  -vga virtio
```
```shell fold title="Скрипт для запуска"
#!/bin/bash

# Значения по умолчанию
DEFAULT_MEM_GB=12
DEFAULT_CORES=4
DEFAULT_GRAPHIC="yes"
DEFAULT_QEMU_PORT=3100

# Пути и MAC-адреса
DISK_PATH="/home/daniil/vm/disks/ubuntu-disk.qcow2"
MAC1="52:54:00:12:34:01"
MAC2="52:54:00:12:34:02"

# Функция отображения справки
show_help() {
    echo "Использование: $0 [ОПЦИИ]"
    echo "Запуск виртуальной машины QEMU/KVM с двумя сетевыми мостами."
    echo ""
    echo "ОПЦИИ:"
    echo "  -m <ГБ>       Оперативная память в гигабайтах (по умолчанию: $DEFAULT_MEM_GB), с.м. 'free -h'"
    echo "  -c <число>    Количество ядер CPU (по умолчанию: $DEFAULT_CORES), с.м. 'lscpu'"
    echo "  -g <yes|no>   Режим отображения: yes — графическое окно, no — текстовая консоль (по умолчанию: $DEFAULT_GRAPHIC)"
    echo "  -h, --help    Показать эту справку"
    echo "   Порт для консоли QEMU по умолчанию: $DEFAULT_QEMU_PORT"
    echo "   Пример:  $0 -m 8 -c 2 -g no     # 8 ГБ, 2 ядра, текстовая консоль"
    echo ""
    echo "MAC-адреса:"
    echo "  Локальная сеть (br-local): $MAC1"
    echo "  Интернет (br-wan):         $MAC2"
    echo "Диск: $DISK_PATH"
}

# Обработка аргументов
MEM_GB=$DEFAULT_MEM_GB
CORES=$DEFAULT_CORES
GRAPHIC=$DEFAULT_GRAPHIC

while [[ $# -gt 0 ]]; do
    case "$1" in
        -m) MEM_GB="$2"; shift 2 ;;
        -c) CORES="$2"; shift 2 ;;
        -g) GRAPHIC="$2"; shift 2 ;;
        -h|--help) show_help; exit 0 ;;
        *) echo "Неизвестная опция: $1"; show_help; exit 1 ;;
    esac
done

# Проверка значений
if ! [[ "$MEM_GB" =~ ^[0-9]+$ ]] || [ "$MEM_GB" -le 0 ]; then
    echo "ОШИБКА: память должна быть положительным целым числом (в ГБ)."
    show_help
    exit 1
fi
if ! [[ "$CORES" =~ ^[0-9]+$ ]] || [ "$CORES" -le 0 ]; then
    echo "ОШИБКА: количество ядер должно быть положительным целым числом."
    show_help
    exit 1
fi
if [[ "$GRAPHIC" != "yes" && "$GRAPHIC" != "no" ]]; then
    echo "ОШИБКА: параметр -g должен быть 'yes' или 'no'."
    show_help
    exit 1
fi

MEM_MB=$((MEM_GB * 1024))

if [ ! -f "$DISK_PATH" ]; then
    echo "ОШИБКА: файл диска $DISK_PATH не найден."
    exit 1
fi

# Выбор режима отображения
if [ "$GRAPHIC" = "yes" ]; then
    DISPLAY_OPTS="-display gtk -vga virtio"
else
    DISPLAY_OPTS="-nographic -serial mon:stdio"
fi

# Всегда открываем монитор через TCP сокет для доступа по SSH
MONITOR_OPTS="-monitor tcp:127.0.0.1:$DEFAULT_QEMU_PORT,server,nowait"

echo "Запуск ВМ с памятью ${MEM_GB}G, ${CORES} ядер, режим: $MODE_STR"
qemu-system-x86_64 \
    -enable-kvm \
    -cpu host \
    -m "$MEM_MB" \
    -smp "$CORES" \
    -drive file="$DISK_PATH",format=qcow2 \
    -netdev bridge,id=net0,br=br-local \
    -device virtio-net-pci,netdev=net0,mac="$MAC1" \
    -netdev bridge,id=net1,br=br-wan \
    -device virtio-net-pci,netdev=net1,mac="$MAC2" \
    $DISPLAY_OPTS \
    $MONITOR_OPTS
```
Если вы хотите запускать ВМ без графического интерфейса (например, по SSH), замените -`display gtk` на `-nographic` и добавьте `-serial mon:stdio`. Для подключения к консоли используйте `virsh` или `ssh` после настройки сети.

Если вы забыли ip адреса, узнать их можно с помощью `arp-scan` после запуска машины:
```bash unfold
sudo arp-scan --interface=br-local
```

# Snapshot
Сделать снимок (snapshot) виртуальной машины в QEMU можно двумя основными способами. Выбор зависит от того, нужно ли сохранить состояние оперативной памяти или только состояние диска. Рассмотрим только полный снимок.
Главное условие — ваш виртуальный диск должен быть в формате **qcow2**.

В консоли QEMU:
Создайте снимок
```bash unfold
(qemu) savevm <имя_снимка>
```
Посмотреть все снимки
```bash unfold
(qemu) info snapshots
```
Восстановиться из снимка
```bash unfold
(qemu) loadvm <имя_снимка>
```
Удалить снимок
```bash unfold
(qemu) delvm <имя_снимка>
```
При восстановлении все несохранённые данные, созданные после снимка, будут потеряны.

***Как войти в консоль QEMU:***
*  в графическом режиме: войти`Ctrl + Alt + 2`, выйти -  `Ctrl + Alt + 1`.
* через ssh:
	1. Запустить машину с пробросом TCP порта:`-monitor tcp:127.0.0.1:3100,server,nowait`
	2. Подключиться к хосту по ssh
	3. Выполнить `telnet localhost 3100` (или `nc localhost 3100`)




