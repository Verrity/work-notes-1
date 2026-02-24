
---
### Теги

### [[ESR|Назад]]
---

Полная пересборка репозитория 1 (не чекаутится на все ветки подряд)
1. `cd <path>/esr-base`
2. `./prepare_working_copy.sh --board wlc30 --update-submodules`
3. `Выставить нужные ветки`
4. `make unconfig; yes | make clean-all; make config-wlc30; make all`

Полная пересборка репозитория 2
	1. `cd <path>/esr-base/build/apps/wlc/wlc-base/Documentation`
	2. `./esr_build.sh c ./esr_build.sh f ./esr_build.sh l ./esr_build.sh a ./esr_build.sh w` (`./esr_build.sh a ./esr_build.sh w` - пересобирает без `cmake`, ломается если есть новые файлы)
