
---
### Теги

### [[Home|Назад]]
### Далее
####
---

#### Патчи
```bash unfold title="Дифф патч"
# Между незакомиченными изменениями
git diff > fix.patch
# Между двумя комитами или ветками
git diff main..feature > feature.patch
git apply fix.patch
```

```bash unfold title="Формат патч (прямо по комитам работает)"
# Патч из 3-х последних коммитов
git format-patch -3
# Между ветками
git format-patch main..feature
# Применить патч с сохранением комитов
git am 0001-some-change.patch
```

* `-p` - выводит `diff` для каждого комита

