
---
### Теги
#esr_rep
### [[Home|Назад]]
### Далее
#### [[Clish]]
---
- Кластер это синхронизация конфигурации, времени, прошивок, общий мониторинг. Фейловер отвечает просто за синхронизацию состояния сервисов.
- В фейловере можно настроить разные устройства, в кластере только одной модели устройства (иначе не соберётся кластер).

`/mnt/config/etc/config/esr/esr_config.json`
#### Загрузить прошивку через u-boot
Если ни одной прошивки на устройстве не осталось, и нельзя выйти из `uboot`, то вот решение
```u-boot folded title="Команды для загрузки прошивки через u-boot (использовать gigabitethernet 1 !!!)"
ipaddr 192.168.1.1
serverip 192.168.1.10
firmware_file wlc30.firmware
saveenv
netboot
```
`ipaddr` - адрес `wlc`; `serverip` - адрес `tftp` сервера ; `firmware_file` - имя файла на сервере ; `saveenv` - ?; `netboot` - загрузить прошивку и запустить устройство

#### Change branch in ESR submodules
switch to 1.36.x
``` unfold
git submodule foreach --recursive 'case $name in webeltex/backend2/web-core) git checkout esr-1.36.x ;; webeltex/frontend/malvic) git checkout esr-master ;; lint) ;; src/components/rrm/rrm-tester) ;; *) git checkout 1.36.x ;; esac'
```

Difference naming in these:
``` unfold
build/apps/webeltex/backend2/web-core
build/apps/webeltex/frontend/malvic
build/apps/webeltex/frontend/malvic/lint
build/apps/wlc/wlc-base/src/components/rrm/rrm-tester
```
OR 
Переходим на нужную ветку в родительском репозитории и делаем:

```bash unfold
git submodule update --remote --recursive --checkout
```
- эта команда сделает checkout на ветки из файлов .gitmodules

В обычном случае, если в сабмодулях установлены ветки, отличающиеся от ветки родителя, при комаде `git submodule update --remote --recursive` будет произведен `merge` веток, указанных в `.gitmodules` родительского репозитория, так как в `.gitmodules` выставлено такое поведение.

#### Обновить тулчейн ESR
При переходе с esr 1.30.х на 1.36.х потребовалось обновить тулчейн, иначе сборка ломалась. 
```bash unfold
sudo /opt/esr-ci/scripts/esr-repo-clone.sh /opt/esr-tools -c update -p tools && sudo /opt/esr-tools/install.sh
```
