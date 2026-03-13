
---
### Теги

### [[System|Назад]]
### Далее
####
---

# Установка

### Шаг 1. Установка необходимых пакетов
Обновите список пакетов и установите QEMU, инструменты для управления мостами и виртуализацией
```unfold
sudo apt update
sudo apt install bridge-utils qemu-kvm libvirt-daemon-system libvirt-clients virtinst qemu-system-x86
```

### Шаг 2. Включение IP-форвардинга
Для работы сетевых мостов и маршрутизации между ними необходимо разрешить пересылку пакетов между интерфейсами
```conf unfold title="/etc/sysctl.conf"
net.ipv4.ip_forward=1
```
```unfold
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
```unfold
sudo netplan apply
```

Шаг 4. Настройка QEMU для использования мостов
QEMU использует вспомогательную программу qemu-bridge-helper для подключения виртуальных машин к мостам. Нужно разрешить использование мостов в конфигурации

Создайте каталог для конфигурации QEMU (если его нет):
```
sudo mkdir -p /etc/qemu
```
```file /etc/qemu/bridge.conf
allow br-local
allow br-wan
```
Установите правильные права:
```
sudo chmod 644 /etc/qemu/bridge.conf
```

Установите SUID-бит на qemu-bridge-helper, чтобы QEMU мог использовать его даже от обычного пользователя (необходимо для запуска ВМ без sudo):
```
sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

Примечание: Путь к qemu-bridge-helper может отличаться в зависимости от версии. Проверьте его командой 
```
find /usr -name qemu-bridge-helper 2>/dev/null
```

Шаг 5. Добавление пользователя в группу kvm
Чтобы иметь доступ к устройствам KVM, добавьте своего пользователя в группу kvm:
```
sudo usermod -aG kvm $USER
```

Шаг 6. Создание образа диска для виртуальной машины
Создайте каталог для дисков и образов (по желанию) и сформируйте qcow2-образ размером 50 ГБ:
```
mkdir -p ~/vm/disks
qemu-img create -f qcow2 ~/vm/disks/ubuntu-disk.qcow2 50G
```

Шаг 7. Установка гостевой ОС
Для установки Ubuntu Server потребуется ISO-образ. Скачайте его, например, в каталог ~/vm/images/. Затем выполните команду запуска QEMU с параметрами:
```
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
-enable-kvm – использовать аппаратную виртуализацию.
-cpu host – передать гостю все возможности хост-процессора.
-m 4096 – 4 ГБ ОЗУ. Указывается в мегабайтах. Например, для выделения 16 ГБ используйте -m 16384, для 32 ГБ — -m 32768.
-smp 2 – Количество ядер. Задаётся числом, например, -smp 8 для 8 виртуальных CPU.
-drive – подключает созданный образ как диск.
-cdrom – подключает ISO для установки.
-netdev bridge – создаёт сетевые интерфейсы, привязанные к мостам br-local и br-wan.
-device virtio-net-pci – задаёт тип сетевой карты и её MAC-адреса.
-display gtk – открывает графическое окно (можно заменить на -nographic для консоли).
-vga virtio – тип видеокарты.

Доступные ресурсы хоста
Убедитесь, что на вашем физическом сервере достаточно свободной оперативной памяти (команда free -h) и свободных ядер CPU (можно посмотреть загрузку через htop или lscpu).

Шаг 8. Настройка сети внутри гостевой системы
После запуска установщика следуйте инструкциям. На этапе настройки сети вам будут доступны два интерфейса:
Первый интерфейс (подключён к br-local, локальная сеть). Настройте его вручную:
IP-адрес: 192.168.1.200 (или любой свободный в вашей локальной подсети)
Маска: 255.255.255.0
Шлюз: оставьте пустым (если не требуется маршрутизация в локальную сеть через хост; обычно не нужен).
Второй интерфейс (подключён к br-wan, интернет). Настройте его на получение адреса по DHCP (автоматически). Тогда виртуальная машина получит IP от роутера и доступ в интернет.
Важно: MAC-адреса, указанные при запуске, должны быть уникальны в вашей локальной сети. При необходимости измените их.
После завершения установки виртуальная машина перезагрузится. Закройте окно QEMU.

Шаг 9. Запуск установленной виртуальной машины
Для последующего запуска (без установочного ISO) используйте ту же команду, но без параметра -cdrom:
```
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

Если вы хотите запускать ВМ без графического интерфейса (например, по SSH), замените -display gtk на -nographic и добавьте -serial mon:stdio. Для подключения к консоли используйте virsh или ssh после настройки сети.

