### Задание 1
#### «CLI‑ассистент: приветствие, валидация и мини‑отчёт о системе»

![цели](image.png)

__________________________________

1) запрос + валидация

```bash
#!/bin/bash

echo "input your name: "
read name

attempt=0

while [ $attempt -le 3 ]; do

if [ -z "$name" ]; then   
   if [ $attempt -eq 3 ]; then
   echo "try again"
   break
   fi
   echo "INPUT YOUR NAME"
   attempt=$((attempt+1))
   read name
else
   echo "hello $name"
   break
fi
done

```

> result

![alt text](image-1.png)
_______________

![alt text](image-2.png)

2) Вывод отчета

```bash
echo -e "\n" "__________MINI_OTCHET__________"
echo -e "\n" "current date: $(date '+%m/%d/%y %H:%M')"
echo -e "\n"  "hostname: $(hostname)"
echo -e "\n"  "system works: $(uptime)"
echo -e "\n"  "disk free: $(df -h /)"
echo -e "\n"  "users: $(w)"
```

> результат

![alt text](image-3.png)

> полный скрипт

```bash
#!/bin/bash

echo "input your name: "
read name
attempt=0
while [ $attempt -le 3 ]; do
if [ -z "$name" ]; then
if [ $attempt -eq 3 ]; then
echo "try again"
exit 1
fi
echo "INPUT YOUR NAME"
attempt=$((attempt+1))
read name
else
break
fi
done

echo "input your group (optional):"
read group

if [ -z $group]; then
group="--"
fi

echo -e "\n" "hello $name from ($group)"
echo -e "\n" "__________MINI_OTCHET__________"
echo -e "\n" "current date: $(date '+%m/%d/%y %H:%M')"
echo -e "\n" "hostname: $(hostname)"
echo -e "\n" "system works: $(uptime)"
echo -e "\n" "disk free: $(df -h /)"
echo -e "\n" "users: $(w)"
```

### Задание 2
#### «Резервное копирование каталога с логированием и ротацией»

![task 2](image-4.png)

_________________

1) принятие аргументов

