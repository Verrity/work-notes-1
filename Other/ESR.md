
---
### Теги
#esr_rep
### [[Home|Назад]]
### Далее
#### [[Pagination]]
#### [[Certificates]]
#### [[Clish]]
#### [[Uboot]]
#### [[Upgrader]]
#### [[Reflection]]
[[Cluster]]
---
- Кластер это синхронизация конфигурации, времени, прошивок, общий мониторинг. Фейловер отвечает просто за синхронизацию состояния сервисов.
- В фейловере можно настроить разные устройства, в кластере только одной модели устройства (иначе не соберётся кластер).

`/mnt/config/etc/config/esr/esr_config.json`


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

#### Copy
При выполнении команды `copy` `file-mgr` создает потоки операций, внутри которых форкаются отдельные процессы - клиенты транспортных протоколов (`TFTP`, `FTP`, `SCP`, `HTTP` и т.д.) из `busybox`, `tnftp`, `openssh`, `wget` и т.д. Все клиенты пропатчены для передачи своего `pid` и текущего прогресса операций через файл, что позволяет управлять ими и отслеживать прогресс передачи данных

#### Build
```unfold
./prepare_working_copy.sh --board wlc30 --update-submodules
```
Выставить нужные ветки (полностью пересобрать)
```unfold
make unconfig; yes | make clean-all; make config-wlc30; make all
```
```unfold
esr-base: make firmware
```