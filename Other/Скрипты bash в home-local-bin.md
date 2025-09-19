
---
### Теги

### Назад
#### [[System]]
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
```shell folded title="sopen.sh"
#!/bin/bash
sudo picocom -b 115200 /dev/ttyUSB3
```

Подключение к `techsupport` по `ssh` (пароль не дефолтный)
```shell folded title="stech.sh"
#!/bin/bash
sshpass -p "password" ssh techsupport@192.168.1.1
```

Копировать в буфер обмена  последовательность команд для начальной настройки `wlc`
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

username techsupport
  password 1234
  do commit
  do confirm
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


