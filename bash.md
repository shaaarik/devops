# Работа с оболочкой
адрес оболочки
```
echo $SHELL # /bin/bash
```
PID процесса
```
echo $$ # 665
```
инфа про процесс
```
ps -p $$
```
небольшой пайп (если есть первый аргумент то вывести его в строке)
```
test -z $1 && echo "Hello, World!" || echo "Hello, $1!" 
```
вывести промт оболочки
```
echo $PS1 #  \[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$
```
изменить промт 
```
PS1="prompt> " # prompt> 
source ~/.bashrc # сбросить промт 
```
звук БИИИП
```
echo -ne '\007' # BEEEP
```
назначит псевдоним команды
```
alias ll='ls -l'
```
вывесть список путей к директориям
```
echo $PATH # /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```
вывести путь к исполняему файлу команды
```
which ls # /usr/bin/ls
```
добавить в пути новую переменную
```
export PATH="${PATH}:<новая директория которую хотите добавить>" # source ~/.bashrc
```
вывести хэш таблицу оболочки
```
hash ls
hash

hits	command
   0	/usr/bin/ls

ls
hash

hits	command
   1	/usr/bin/ls
```
очистить хэш таблицу
```
hash -r
```
добавить команду
```
hash <имя_команды> 
```
удалить команду
```
hash -d <имя_команды>
```

вывести последней код завершения последней выполненной команды
```
$ bash -c "exit 2"
$ echo $?
2

$ bash -c "exit 2"
$ if [[ $? -eq 0 ]]; then echo "Success"; else echo "Error"; fi
Error
```
показывает информацию о исполняемом файле текущего shell-процесса.
```
ls -l /proc/$$/exe
lrwxrwxrwx 1 ubuntu ubuntu 0 Sep 10 12:09 /proc/1190/exe -> /usr/bin/bash 
```
Переменная $! содержит идентификатор последнего запущенного фонового процесса
```
$ echo $!
```
# Работа со строками и переменными

ввесте 2 переменные и потом сложить их и вывести математический результат
```
ARG1=5
ARG2=7
$ echo $ARG1+$ARG2 | bc
12 
```

выводит строку буквально, без подстановки переменной
```
$ echo '$JAVA_HOME'
$JAVA_HOME
```
подставляет значение переменной
```
$ echo "$JAVA_HOME"
/usr/lib/jvm/java-17-openjdk-amd64 
```
математические операции со строками
```
$ RESPONSE_CODE=400
$ echo $RESPONSE_CODE
400
$ RESPONSE_CODE=400+4
$ echo $RESPONSE_CODE
400+4
$ RESPONSE_CODE=$((400+4))
$ echo $RESPONSE_CODE
404 
```
создать переменную с выводом кода результата HTTP-запроса
```
$ RESPONSE_CODE=$(curl -w '%{response_code}' -I -s -o /dev/null http://practicum.yandex.ru)
$ echo $RESPONSE_CODE
302 
```
Объявить переменную только для чтения 
```
$ declare -r MY_VARIABLE="Hello, World!"
$ MY_VARIABLE="Hello, World?"
bash: MY_VARIABLE: readonly variable

$ readonly MY_VARIABLE="Hello, World!"
$ MY_VARIABLE="Hello, World?"
bash: MY_VARIABLE: readonly variable
```

примеры использования замены в строке через sed и awk
```
$ echo "Hello!" | sed 's/Hello/Hi/'
Hi!


$ echo "Hello! Hello!" | sed 's/Hello/Hi/g'
Hi! Hi!


$ echo "Hello1! Hello2!" | awk '{print $1}'
Hello1!
$ echo "Hello1! Hello2!" | awk '{print $2}'
Hello2!


$ echo 'Hello1!Hello2!' | awk -F'!' '{print $1}'
Hello1

$ echo -e 'Hi1!Hi2!\nHello1!Hello2!'
Hi1!Hi2!
Hello1!Hello2!
$ echo -e 'Hi1!Hi2!\nHello1!Hello2!' | awk -F'!' '/Hello/ {print $1}'
Hello1
```
использование скрипта для awk
```
$ cat script.awk
BEGIN {}
{ print $1; }
END {}
$ echo "Hello1! Hello2!" | awk -f script.awk
Hello1! Hello2!


$ cat script.awk
BEGIN {
    # Словарь для хранения IP-адресов
    # ips - ассоциативный массив не хранит дубликаты
}
# Обработка строк, которые начинаются с IP-адреса
$1 ~ /^(([0-9]+\.)|([0-9a-f]+:))/ && NF = 12 {
    # Добавляем IP-адрес в словарь
    if ($10 > 200) {
        ips[$1] = 1;
    }
}
END {
    # Выводим IP-адреса
    for (ip in ips) {
        print ip
    }
}

$ awk -f script.awk access.log
ffff:2f0:d13:2e19:10b:28f0:e17e:0
ffff:2f0:d13:2b03:10b:28f0:209e:0
ffff:2f0:d13:2a91:10b:28f0:5a8f:0
ffff:2f0:d13:2a8a:10b:28f0:5d04:0
ffff:6b8:c0f:4882:10b:28f0:e727:0
```

ввод в оболочку по строкам
```
$ cat <<EOF
> Hello,
> World!
> EOF
Hello,
World!
```

ввод в файл по строкам
``` 
$ cat <<EOF > file.txt
> Hello,
> World!
> EOF
$ cat file.txt
Hello,
World!
```
ввод в файл по строкам через "cat -"
```
$ cat - > file.txt
Hello,
World!
^D
$ cat file.txt
Hello,
World!
```
``` 
$ name="World"
$ cat <<< "Hello, $name!"
Hello, World!
```
# Массивы
объявить массив и вывести результат по индексу
```
$ declare -a MY_ARRAY
$ MY_ARRAY[0]="Hello"
$ MY_ARRAY[1]="World"
$ echo ${MY_ARRAY[0]} ${MY_ARRAY[1]}!
Hello World! 
```
объявить массив одновременно с присвоением значений
```
$ MY_ARRAY=("Hello" "World")
$ echo ${MY_ARRAY[0]}, ${MY_ARRAY[1]}!
Hello, World!
```
поменять символ разделения в массиве
```
#!/bin/bash
# Пусть на вход поступает набор слов, разделённых двоеточием
line="home-directory:bash-directory"
# Переопределим IFS
IFS=":"
# Создадим массив
tokens=($line)
echo "${tokens[0]}"  # Выводит "home-directory"
echo "${tokens[1]}"  # Выводит "bash-directory"
```
поменять символ разделения в массиве на запятую, объявить массив через строку и вывести 9ый элемент
```
IFS=","
month=",January,February,March,April,May,June,July,August,September,October,November,December"
MONTHS=($month)
echo ${MONTHS[9]}
September
```
# Условия

проверить существует ли файл, если нет, тогда создать файл
```
$ test -f /tmp/my_file.txt
$ if [[ $? -eq 1 ]]; then touch /tmp/my_file.txt; fi
```
-e выход в случае ошибки, -u выход при использовании неопределенных переменных, -o pipefail - учитывать ошибки в пайпах
```
set -euo pipefail

if [ -z "${INPUT}" ] #если есть INPUT
then 
  echo "Hello, World!"
else 
  echo "Hello, ${INPUT}!"
fi
```
квадратные скобки заменяют команду test
```
var1=7
if test "$var1" -lt 10; then
  echo "$var1 меньше 10."
fi

var1=7
if [ "$var1" -lt 10 ]; then
  echo "$var1 меньше 10."
fi
```
```
string1="Hello"

if [[ $string1 == *el* ]]; then
    if [[ $string1 == *llo* ]]; then
        echo "Строка '$string1' содержит подстроку 'el' и подстроку 'llo'"
    fi
    echo "Строка '$string1' содержит подстроку 'el'"
else
    echo "Строка '$string1' не содержит подстроки 'el' и 'llo'"
fi

```
проверка является ли пользователь root или нет
```
if [ $(id -u) -eq 0 ]; then
    echo "User is root"
else
    echo "User is not root"
fi
```
сравнение массивов
```
array1=("a" "b" "c")
array2=("a" "b" "c")
if [ "${array1[*]}" = "${array2[*]}" ]; then
    echo "Массивы равны"
else
    echo "Массивы не равны"
fi
```
использование case
```
animal="хомяк"

case $animal in
    хомяк)
        echo "Это хомяк";;
    питон)
        echo "Это питон";;
    верблюд)
        echo "Это верблюд";;
    *)
        echo "Неизвестное животное";;
esac
```
# Циклы
цикл из диапазона
```
for i in {1..5}
do
    echo $i
done
```
перебоор массива
```
array=("python" "camel" "cherry")
for animal in "${array[@]}"
do
    echo $animal
done
```
Основное отличие между ${array[*]} и ${array[@]} в том, как они обрабатываются в контексте кавычек:

Если ${array[*]} внутри кавычек, все элементы массива будут рассматриваться как одна строка.
Если ${array[@]} внутри кавычек, каждый элемент массива будет рассматриваться как отдельная строка. Также полезно, когда элемент массива содержит пробел в значении.
```
echo "list in quotes"
for arg in "$@"; do
    echo "Аргумент: $arg"
done
echo "string in quotes"
for arg in "$*"; do
    echo "Аргумент: $arg"
done


list in quotes
Аргумент: arg1
Аргумент: arg2
Аргумент: arg3
string in quotes
Аргумент: arg1 arg2 arg3
```


Выведет на экран все «обычные» файлы в текущей директории, а затем для каждого файла выведет его размер
```
for file in *
do
  echo "Файл: $file"
  if [ -f "$file" ]
  then
    size=$(du -h "$file" | cut -f1)
    echo "Размер: $size"
  fi
done
```
цикл с инкрементацией
```
i=1
while [ $i -le 10 ]
do
    echo $i
    i=$((i+1))
done

i=1
until [ $i -gt 10 ]
do
    echo $i
    i=$((i+1))
done
```
однострочный цикл
```
while true; do for i in {1..10}; do [ $i -eq 5 ] && continue; echo $i; done; exit; done
```
# Команды

вывести форматированную строку текущей даты
```
$ echo $(date +%Y-%m-%d)
2021-09-10

$ printf "Today is %(%A, %d %B %Y)T\n"
Today is Friday, 10 September 2021 
```
используется для завершения процессов по регулярному выражению, которые были запущены более 10 дней назад
```
$ killall -o 10d -r <regexp имен процессов>
$ ps --no-headers -eo pid,etimes,cmd | awk '$2 > 864000 {print $1}' | xargs kill
$ find /proc -maxdepth 1 -type d -regex '.*/[0-9]+' -mtime +10 -exec basename {} \; | xargs kill -9
```
декодирует base64-закодированный SSH приватный ключ и сохраняет его в файл
```
base64 -d - <<< ${SSH_PRIVATE_KEY_BASE64} > ~/.ssh/id_rsa 
```

# Работа с файлами

инфа про файл
```
file hello.sh
```
найти файл содержащим слово pipe
```
find . -name "*pipe*"
```
найдёт все файлы каналов в текущей директории
```
find . -type p 
```
файлы размером больше одного мегабайта, но меньше 10 мегабайт
```
find /var/log -type f -size +1M -size -10M 
```
назначить псевдоним nginx_top, который выводит самый часто встречающийся ip адрес в логе nginx
```
alias nginx_top="awk '{print $1}' /var/log/nginx/access.log | uniq -c | sort -r | head -1" # 9 80.76.51.50
```
Если вместо списка файлов передать -, то cat будет читать данные со стандартного потока ввода
```
cat - 
```
-i для игнорирования регистра и -r для рекурсивного поиска в директории
```
sudo grep error -ir /var/log/ // 
```
find . - ищет файлы в текущей директории и поддиректориях
-name "*.bak" - фильтрует файлы по расширению .bak
-exec rm {} \; - для каждого найденного файла выполняет команду rm
  {} - заменяется на имя найденного файла
  \; - обозначает конец команды
```
$ find . -name "*.bak" -exec rm {} \;
```
выполняет параллельное удаление всех файлов с расширением .bak в текущей директории и поддиректориях.
```
$ find . -name "*.bak" | xargs rm
$ find . -name "*.bak" | xargs -P 4 rm
```

# Скрипты
скрипт с выбором ответа
```
#!/bin/bash

Text='Please enter your choice(1-3): '
options=("Gopher" "Camel" "Python" )

select choice in "${options[@]}"
do
    case $choice in
        "${options[0]}")
            echo "Ваше любимое животное Gopher"
            break
            ;;
         "${options[1]}")
            echo "Ваше любимое животное Camel"
            break
            ;;
         "${options[2]}")
           echo "Ваше любимое животное Python"
           break:
            ;;
        *) echo "Вы не сказали, какое у вас любимое животное =(";;
    esac
done
```
фунцкии
```
function sum {
    a=$1
    b=$2
    result=$((a + b))
    return $result
}

sum 5 10
sum_result=$?
echo "Сумма: $sum_result"
```
& запускает процесс в фоне
```
function my_function {
    sleep 5
    echo "Функция завершена"
}

my_function &
echo "Скрипт продолжает выполнение"
```
пример скрипта на bash с флагами 
При помощи -- в парсерах команной строки принято обозначать, что больше нет аргументов, которые начинаются с -
```

show_help() {
  echo "Использование: script.sh [опции]"
  echo "Опции:"
  echo "  --host              Показать информацию о хосте"
  echo "  --test <аргумент>   Описание опции test, какие аргументы принимает"
  echo "  -h, --help          Показать help"
}
# Объявление длинных ключей
OPTIONS=$(getopt -o "h" -l "help,host,test:" -- "$@")
# Вывод help если не передали значения
if [[ $# -eq 0 ]]; then
  show_help
  exit 0
fi
# Парсинг аргументов
eval set -- "$OPTIONS"
# Обработка аргументов
while true; do
  case "$1" in
    -h|--help)
      echo "Показать справку"
      show_help
      exit 0 ;;
    --host)
      echo "Обработка --host"
      shift ;;
    --test)
      echo "Обработка --test с аргументом $2"
      shift 2 ;;
    --)
      shift
      break ;;
    *)
      echo "Неправильный аргумент"
      exit 1 ;;
  esac
done

rm -- -r //  удалить в командной строке файл, который называется -r
```