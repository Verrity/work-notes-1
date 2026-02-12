
---
### Теги

### [[Home|Назад]]
### Далее
####
---
#### Символы
* <font color="#f79646">`@` - синоним `HEAD`</font>
* <font color="#f79646">`..` - Показывает изменения между комитами.</font>
Используется для прямого сравнения веток или коммитов.
```bash unfold
git log A..B  # Коммиты, которые есть в A но нет в B
git diff A..B # Разница снимков между А и В
```
* <font color="#f79646">`...` - Показывает симметричную разность между A и B (все что есть в одной ветке, но нет в другой).</font>
Применяется, когда у веток есть общий предок `merge base`. 
```bash unfold
git diff main..feature # Покажет, как feature отличается общего предка с main (как при ревью)
git diff A...B # Разница между каждым и общим предком
```
```bash unfold
git log A...B # Комиты, различающиеся между ветками
```
* <font color="#f79646"><font color="#f79646"></font>`^` - переход к родителю коммита. </font>
HEAD^ - первый родитель
HEAD\^2 - второй родитель (у merge-коммита)
HEAD^^ = HEAD^1\^1 = родитель родителя
HEAD^^^ = три поколения назад
* <font color="#f79646">`~` - переход на N поколений назад по первому родителю</font>
HEAD~1 = HEAD\^1
HEAD~2 = первый родитель первого родителя
HEAD~3 = три шага назад

**Через `^` можно выбрать конкретного родителя (`^2`), а `~` всегда идет по первой ветке**
* <font color="#f79646">`!` - сокращение диапазона, аналог `--not <commit>^`. Например:</font>

```bash unfold
git show <hash>         # Патч, который внес комит
git diff <hash>^ <hash> # То же самое (отличие комита от его родителя)
git log <hash>^!        # Лог всего что не родитель (только этого коммита, отдельно от дерева)
```
Это сокращение для 
```bash unfold
git diff <commit>^ <commit>
то же самое, что и
git log A^! или git range-diff A^! B^!
```
* <font color="#f79646">Комбинирование символов</font>
`HEAD~3^2` - у третьего предка `HEAD` взять второго родителя
`A^!` <=> `A^..A` - диапазон только одного родителя
`A^@` - все родители комита `A` (полезно при merge анализе)
`A^..B` - все комиты после A, включая B.

#### How to do
1. <u>Посмотреть разницу межу изменениями комитов (не учитывая все дерево между ними, только их самих)</u>
```bash unfold
git range-diff <hash1>^! <hash2>^!
```
#### Инструмент git range-diff
Используется для сравнения двух наборов патчей, а не просто состояний. Используется, когда коммиты были переписаны `rebase`, `cherry-pick`, `amend` и нужно понять, что изменилось до и после (сравнение версий ветки в gitlab).
```bash unfold 
git range-diff oldbase..oldhead newbase..newhead
```
Строит два списка патчей из 1 и 2 диапазонов, сопоставляет их по содержимому (не по хэшам), показывает какие патчи совпали, а какие были изменены.
Пример сравнения патчей двух комитов относительно их родителей:
```bash unfold
git range-diff 298452e5532d8c91ff189420e5115e2b1337d9cc^..298452e5532d8c91ff189420e5115e2b1337d9cc cdf79b5b495e53e81bbed453ae44a4dfbe6bfa25^..cdf79b5b495e53e81bbed453ae44a4dfbe6bfa25
```

```sh fold title="Скрипт git_range_diff.sh"
#!/bin/bash
# Скрипт похож на "git range-diff", но позволет сравнить диапазон комитов как один патч
# c его родительским комитом в качестве диапазона с другим таким же диапазоном. Обычный
# range-diff сравнивает диапазоны покомитно, но количество и качество комитов не всегда
# совпадает, в таком случае вообще не понятно, что выводит range-diff.
# Вы вводите диапазон "first_commit last_commit" - он сравнивает
# "first_commit_parent" с "[first_commit..last_commit]" как один диапазон.
if [ $# -ne 4 ]; then
    echo "Usage: $0 <1_first_commit> <1_last_commit> <2_first_commit> <2_lase_commit>"
    echo "Example: $0 62b8cf3 1412a30 5b5da72 c30db82"
    exit 1
fi
# комиты
D1_FIRST=$1
D1_LAST=$2
D2_FIRST=$3
D2_LAST=$4
# создаем и запоминаем ветки
CURRENT_BRANCH_OR_COMMIT=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || git rev-parse HEAD)
D1_TMP_BRANCH="tmp-range-diff-old-$$"
D2_TMP_BRANCH="tmp-range-diff-new-$$"
# очистка при выходе
cleanup() {
    git branch -D $D1_TMP_BRANCH 2>/dev/null || true
    git branch -D $D2_TMP_BRANCH 2>/dev/null || true
    git checkout $CURRENT_BRANCH_OR_COMMIT 2>/dev/null || true
}
trap cleanup EXIT
# создаем временные ветки
git checkout -b $D1_TMP_BRANCH $D1_LAST
git checkout -b $D2_TMP_BRANCH $D2_LAST
# сквошим старый набор
git checkout $D1_TMP_BRANCH
git reset --soft ${D1_FIRST}^
git commit -m "SQUASHED: $D1_FIRST..$D1_LAST"
D1_TARGET=$(git rev-parse HEAD)
# сквошим новый набор
git checkout $D2_TMP_BRANCH
git reset --soft ${D2_FIRST}^
git commit -m "SQUASHED: $D2_FIRST..$D2_LAST"
D2_TARGET=$(git rev-parse HEAD)
# смотрим diff
git range-diff $D1_TARGET^..$D1_TARGET $D2_TARGET^..$D2_TARGET
# cleanup и возврат
cleanup
exit 0
```

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

# Из 3-х комитов одновременно
git format-patch -3 --stdout > pat
```

* `-p` - выводит `diff` для каждого комита

#### Команды
```bash unfold title"Посмотреть файл в другой ветке не переходя на нее"
git show <branch>:<filepath>
```