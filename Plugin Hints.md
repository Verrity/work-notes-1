
# HK Code Block
* Add title

reset
* `collapse`. По умолчанию выключен - включить `collapse или collapse:true`
* `title`. По умолчанию выключен. Включить `title:"<title>"`

Примеры:
```c title:"Заголовок текстового блока (включен)" collapse:true
printf("Hello world\n");
// Комментарий
printf("Hello world\n");
printf("Hello world\n");
printf("Hello world\n");
printf("Hello world\n");
printf("Hello world\n");
```
.
```python title:"Выделить строки кода" highlight:"1-2,4"
print("line 1")
print("line 2")
print("line 3")
print("line 4")
print("line 5")
```
.
```c collapse
print("line 2")
```
seteRrrte
```bash lin
mkdir test
```






.
```python unfold
print("line 111")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
```
.
```python collapse:false
print("line 1")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
```
.
```python collapse:true
print("line 1")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
print("line 1")
```