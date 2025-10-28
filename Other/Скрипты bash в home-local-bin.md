
---
### Теги

### [[System|Назад]]
### Далее
####
---
Копирование конфигурации `wlc30.firmware`на validator
```shell folded title="scopy.sh"
#!/bin/sh

USER_NAME='daniil.levin'
USER_IP='192.168.1.2'
WLC_USER='admin'
WLC_PASS='admin'
WLC_IP='192.168.1.1'
TFTP_DIR='/srv/tftp'
TFTP_PORT=69
FW_NAME='wlc30.firmware'
FW_NAME_OLD='wlc30.firmware.old'

fw=$TFTP_DIR/$FW_NAME
old_fw=$TFTP_DIR/$FW_NAME_OLD

# download new fw
curl -s -o $fw https://validator.eltex.loc/checkerusers/$USER_NAME/files/wlc30.firmware

if [ -f $fw ]; then
  # Сравниваем FW на эквивалентность
  # if [[ [-f $old_fw] && cmp -s $fw $old_fw ]]; then
  #   echo "Fail! This firmware already exists!" 
  #   rm $fw
  # else
 
  # Отправляем fw на WLC
  sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "copy tftp://$USER_IP:$TFTP_PORT:/$FW_NAME system:firmware" 
  sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "boot system inactive"
  sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "reload system"
else
  echo "$fw not found!"
fi

# rename old FW
if [ -f $fw ]; then
  mv $fw $old_fw
fi
```

```python title="scopy.py - умный скрипт загрузки"
#!/usr/bin/env python3
import argparse
import subprocess
import sys
from pathlib import Path
from ipaddress import IPv4Address
from urllib.parse import urlparse
from datetime import datetime
import os

# === Default settings ===

default_wlc_user_settings = {
    'user': "admin",
    'password': "admin"
}

default_wlc_settings = {
    'user_profile': default_wlc_user_settings,
    'ip': "192.168.1.1"
}

default_host_settings = {
    'user': "daniil",
    'ip': "192.168.1.10",
}

default_source_settings = {
    'port': 69,
    'tftp_dir': "/srv/tftp", # For path check only
    'download_dir': "/srv/tftp", # For firmware download (must be in tftp_dir)
    'firmware_link': "https://validator.eltex.loc/checkerusers/daniil.levin/files/wlc30.firmware",
    'firmware_name': "wlc30.firmware"
}

default_settings = {
    'wlc': default_wlc_settings,
    'host': default_host_settings,
    'source': default_source_settings
}

# === Функции ===

def setup_argparse() -> argparse.ArgumentParser:
    """Настраивает и возвращает парсер аргументов."""
    parser = argparse.ArgumentParser(
        description="Upload firmware to a WLC device via SSH and TFTP.",
        formatter_class=argparse.RawTextHelpFormatter,
        allow_abbrev=False
    )
    # Взаимоисключающие аргументы для источника firmware
    fw_group = parser.add_mutually_exclusive_group()
    fw_group.add_argument("positional", nargs="?", 
                         help="Firmware file name or direct download link (same as using -f).")
    fw_group.add_argument("-f", "--file", metavar="FILE", 
                         help="Specify firmware file located in the TFTP directory.")

    # Взаимоисключающие аргументы для IP назначения
    ip_group = parser.add_mutually_exclusive_group()
    ip_group.add_argument("-i", "--ip-subnet", metavar="N", 
                         help="Set last octet of default IP. Example: -i 22 → 192.168.1.22")
    ip_group.add_argument("-ii", "--ip-and-subnet", metavar="X.Y", 
                         help="Set last two octets. Example: -ii 22.17 → 192.168.22.17")
    ip_group.add_argument("-I", "--ip-address", metavar="A.B.C.D", 
                         help="Set full destination IP. Example: -I 192.168.2.100")
    
    parser.epilog = (
        "Examples:\n"
        "  scopy\n"
        "  scopy wlc30-test.firmware\n"
        "  scopy https://eltex-validator/wlc30-38923632.firmware\n"
        "  scopy -i 2 -f 1.30.1.firmware\n"
        "  scopy -I 19.22.21.31 -f ../test.firmware"
    )
    
    return parser

def ip_from_octets(base_ip: IPv4Address, octet1=None, octet2=None, octet3=None, octet4=None):
    """Создает IP-адрес на основе базового с заменой октетов."""
    octs = list(base_ip.packed)
    if octet1 is not None:
        octs[0] = octet1
    if octet2 is not None:
        octs[1] = octet2
    if octet3 is not None:
        octs[2] = octet3
    if octet4 is not None:
        octs[3] = octet4
    return IPv4Address(bytes(octs))

def args_resolve_destination_ip(args, parser: argparse.ArgumentParser) -> IPv4Address:
    """Определяет IP-адрес назначения на основе аргументов."""
    try:
        if args.ip_address:
            return IPv4Address(args.ip_address)
        elif args.ip_and_subnet:
            oct3, oct4 = map(int, args.ip_and_subnet.split("."))
            base_ip = IPv4Address(default_wlc_settings['ip'])
            return ip_from_octets(base_ip, octet3=oct3, octet4=oct4)
        elif args.ip_subnet:
            oct4 = int(args.ip_subnet)
            base_ip = IPv4Address(default_wlc_settings['ip'])
            return ip_from_octets(base_ip, octet4=oct4)
        else:
            return IPv4Address(default_wlc_settings['ip'])
    except Exception as e:
        parser.error(f"Invalid IP specification: '{e}'.")

def download_firmware(url: str, output_path: Path) -> bool:
    """Скачивает firmware по URL."""
    print(f"Downloading firmware ...\n", end="", flush=True)
    
    try:
        # Используем curl с прогресс-баром
        result = subprocess.run(
            ["curl", "-L", "--progress-bar", "-o", str(output_path), url],
            capture_output=True,
            check=True
        )
        print("Download completed.")
        return True
    except subprocess.CalledProcessError as e:
        print(f"Download failed: '{e}'.")
        return False

def get_file_last_modified_time(path: str) -> str:
    try:
        return datetime.fromtimestamp(os.path.getmtime(path))
    except:
        return ""

def resolve_firmware_source(fw_arg: str, parser: argparse.ArgumentParser) -> dict:
    """Определяет источник firmware и возвращает информацию о нем."""
    fw_info = {
        'name': None,
        'path': None,
        'source_desc': None,
        'last_modified_time': None,
        'tags': []
    }

    if not fw_arg:
        # -f не указан
        # Скачивание firmware по умолчанию
        fw_info['name'] = default_source_settings['firmware_name']
        fw_info['path'] = Path(default_source_settings['download_dir']) / fw_info['name']
        fw_info['tags'] = ["Remote", "Default"]
        
        if not download_firmware(default_source_settings['firmware_link'], fw_info['path']):
            parser.error("Failed to download default firmware.")

    else:
        # -f указан
        parsed = urlparse(fw_arg)
        if parsed.scheme in ("http", "https"):
            # Скачивание прошивки с указанного url
            fw_info['name'] = default_source_settings['firmware_name']
            fw_info['path'] = Path(default_source_settings['download_dir']) / fw_info['name']
            fw_info['tags'] = ["Remote", "Specified"]
            fw_info['source_url'] = f"{default_source_settings['firmware_link']}"
            
            if not download_firmware(fw_arg, fw_info['path']):
                parser.error(f"Failed to download firmware from '{fw_arg}'.")

        else:
            # Разрешение пути локального файла
            fw_path = Path(fw_arg).expanduser().resolve()
            if not fw_path.exists():
                parser.error(f"Specified firmware file '{fw_path}' does not exist.")
            
            try:
                fw_rel = fw_path.relative_to(default_source_settings['tftp_dir'])
            except ValueError:
                parser.error(f"Firmware path '{fw_path}' is outside TFTP directory '{default_source_settings['tftp_dir']}'.")
            
            fw_info['name'] = str(fw_rel)
            fw_info['path'] = fw_path
            fw_info['tags'] = ["Local", "Specified"]
    
    # Получаем время модификации файла
    if fw_info['path'] and fw_info['path'].exists():
        fw_info['last_modified_time'] = datetime.fromtimestamp(os.path.getmtime(fw_info['path']))
    
    return fw_info

def execute_ssh_commands(dest_ip: IPv4Address, fw_name: str):
    """Выполняет команды SSH на целевом устройстве."""
    password = default_wlc_settings['user_profile']['password']
    user_name = default_wlc_settings['user_profile']['user']
    host_ip = default_host_settings['ip']
    tftp_port = default_source_settings['port']

    ssh_base = ["sshpass", "-p", password, "ssh", 
                f"{user_name}@{dest_ip}", "-T", "-o", "StrictHostKeyChecking=no"]
    
    commands = [
        f"copy tftp://{host_ip}:{tftp_port}:/{fw_name} system:firmware",
        "boot system inactive", 
        "reload system"
    ]
    
    for i, cmd in enumerate(commands, 1):
        print(f"Executing {i}/{len(commands)}: {cmd} ...", end=' ', flush=True)
        sshpass_args = ssh_base + [cmd]
        try:
            result = subprocess.run(
                sshpass_args, 
                capture_output=True, 
                text=True,
                timeout=30  # Таймаут 30 секунд на команду
            )
            
            # if result.returncode != 0:
            #     print(f"  [!] Command failed with code '{result.returncode}'.")
            #     if result.stdout:
            #         print(f"      stdout: {result.stdout.strip()}")
            #     if result.stderr:
            #         print(f"      stderr: {result.stderr.strip()}")
            #     return False
            # else:
            #     print("[!] Done.")
            print("Done.")
            
        except subprocess.TimeoutExpired:
            print("  [!] Command timed out.")
            return False
        except Exception as e:
            print(f"  [!] Command failed: '{e}'.")
            return False
    
    return True

def print_summary(dest_ip: IPv4Address, fw_info: dict):
    """Выводит сводную информацию о процессе."""
    host_ip = default_host_settings['ip']
    tftp_port = default_source_settings['port']
    # download_dir = default_source_settings['download_dir']
    # tftp_dir = default_source_settings['tftp_dir']
    
    print("\n======= Firmware Upload =======")
    print(f"WLC IP           : {dest_ip}")
    print(f"Firmware File    : {fw_info['path']}")
    print(f"Tags             : {', '.join(fw_info['tags'])}")

    if fw_info['tags'] == ["Remote", "Specified"]:
        print(f"Source           : {fw_info['source_url']}")
    
    if fw_info['last_modified_time']:
        print(f"Last modified    : {fw_info['last_modified_time'].strftime('%Y-%m-%d %H:%M:%S')}")
    else:
        print(f"Last modified    : Unknown")
        
    print(f"TFTP Server      : {host_ip}:{tftp_port}")
    # print(f"TFTP Directory   : {tftp_dir}\n")
    # print(f"TFTP Directory   : {download_dir}\n")
    print("")

# === Основная функция ===
def main():
    """Основная функция скрипта."""
    
    # Парсинг аргументов
    parser = setup_argparse()
    args = parser.parse_args()
    fw_arg = args.file or args.positional
    
    # Определение IP назначения
    dest_ip = args_resolve_destination_ip(args, parser)
    
    # Определение источника firmware
    fw_info = resolve_firmware_source(fw_arg, parser)
    
    # Выполнение SSH команд
    if not execute_ssh_commands(dest_ip, fw_info['name']):
        print(f"\nFirmware upload failed!")
        return 1
    
    # Вывод сводки
    print_summary(dest_ip, fw_info)
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

Копирование конфигурации по локальному имени
```shell folded title="scopy.sh.once"
#!/bin/sh

USER_NAME='daniil.levin'
USER_IP='192.168.1.2'
WLC_USER='admin'
WLC_PASS='admin'
WLC_IP='192.168.1.1'
TFTP_DIR='/srv/tftp'
TFTP_PORT=69
FW_NAME=$1

fw=$TFTP_DIR/$FW_NAME

if [ -f $fw ]; then
    # Отправляем fw на WLC
    sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "copy tftp://$USER_IP:$TFTP_PORT:/$FW_NAME system:firmware" 
    sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "boot system inactive"
    sshpass -p "$WLC_PASS" ssh $WLC_USER@$WLC_IP -T "reload system"
else
  echo "$fw not found!"
fi
```

Перезагрузка ветки (удаление с переключением на 1.30.x, fetch, pull, и скачивание заного)
```shell folded title="reload.sh"
#!/bin/bash
branch=$(git branch --show-current)
if [ "$branch" != "1.30.x" ]; then
        git switch 1.30.x && git fetch && git pull && git branch -D ${branch} && git fetch && git pull && git switch ${branch}
else
  git fetch
  git pull 
        echo "Already 1.30.x!"
fi
```

Открытие `picocom` соединения
```shell folded title="popen.sh"
#!/bin/bash
sudo picocom -b 115200 /dev/ttyUSB$1
```

Подключение к `techsupport` по `ssh` (пароль не дефолтный)
```shell folded title="stech.sh"
#!/bin/bash
sshpass -p "password" ssh techsupport@192.168.1.1
```

Копировать в буфер обмена  последовательность команд для начальной настройки `wlc`
- Настраивать `ntp` только если установлен сервер
```shell folded title="rcopy_std_config.sh"
#!/bin/sh
echo "
copy system:factory-config system:candidate-config
commit
confirm
config

username admin
  password admin 
exit

no ntp broadcast-client enable
ntp enable
ntp server 192.168.1.10
exit

interface gigabitethernet 1/0/1
no switchport access vlan
exit

username techsupport
  password 1234
  do commit
  do confirm
  mode techsupport
  password password
exit
tech-support login enable

syslog console
  severity error
exit

wlc
  ap-profile default-ap
    services
      ip http server
      ip ssh server
    exit
  exit
exit

exit
commit
confirm
" | xclip -selection clipboard
```


