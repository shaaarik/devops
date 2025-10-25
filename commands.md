/boot/grub/grub.cfg // конфиг загрузчика
/boot/vmlinuz // ядро
/boot/initrd.img //образ ФС
sudo update-grub // обновить GRUB (/etc/default/grub)
uname -r // версия ядра
/proc/cmdline // Строка параметров загрузки ядра (собираются отсюда /boot/grub/grub.cfg)
/sys/firmware/dmi/tables/DMI // версия прошивки BIOS 
newgrp docker // применить изменения на группу
/etc/security/limits.conf // ограничения
echo $SHELL // оболочка
cat no_such_file.txt > error.txt 2>&1 // поток вывода и ошибок
sudo lsof /dev/ptmx //все запущенные pts от ptmx

tmux 
ctrl+B + C // копировать сессию
nohup sleep 100 & // запустить в фоне
fg // вернуться к процессу в фоне
bg //продолжить выполнение 
jobs // посмотреть список

tee dmesg.txt // писать и в консоль и в файл
kill -l // список сигналов для процессов
wc //считать строки, слова и байты
 
ps -o sid,pgid,ppid,pid,tty,cmd //список телетайпов с группами
cat /proc/self/environ // окружение процесса
which ls // посмотреть путь до файла исполения
type ls // посмотреть тип команды
export MY_VAR=2234 // экспорт в окружения
echo 'export VARNAME="Value"' >> ~/.bashrc // постоянно закрепит переменнуб даже после перезапуска
source ~/.bashrc // перечитать конфигурацию
sudo lsblk // список блочных устройств
sudo fdisk -l /dev/vdb // инфа про диск
sudo fdisk /dev/vdb // начать размечать диск
sudo pvcreate /dev/vdb1 /dev/vdb2 // создать физический том
sudo vgcreate vg1 /dev/vdb1 // создать группу томов
sudo lvcreate -n lv1 -l 50%FREE vg1 // создать логический том
sudo mkfs.ext4 /dev/vg1/lv // разметить на ext4
sudo tune2fs -m 0 /dev/vg1/lv1 //отключаем резервирование под root
findmnt // посмотреть список примонтированных устройств

guestmount -a slitaz-4.0-base.iso -m /dev/sda mount2/ // монтировать iso
archivemount linux-0.01.tar.gz /mnt/mount1/ // монтировать другой архив
umount /mnt/fs // отмонтировать
fusermount -u /mnt/mount1/ // отмонтировать без повышения привелегий

sudo mount -o remount,ro /dev/vg1/lv1 /mnt/mount3 // монтировать lv1 с опцией только на чтение

sudo mkdir -p /mnt/mount{1,2} // создать 2 папки если еще не созданы

sudo bash -c 'echo "/dev/vg1/lv1 /mnt/mount1 ext4 defaults 0 0" >> /etc/fstab' // запись в конфиг чтобы повторилось после перезапуска
sudo mount -a // перечитать /etc/fstab

whereis mkfs.ext3 // узнать путь до исполняемого файла
find /usr/sbin/ -inum 10341212 // искать по иноде 

touch file.txt // Создание текстового файла
ln -s /path/to/file link // создание мягкой ссылки
ln /path/to/file link // создание хардлинка
mknod device c 1 3 // создание пустого символьного устройства
mknod device b 1 3 // создание пустого блочного устройства
mkfifo pipe // создание пайпа
stat // проверить тип файла

cat /sys/block/sda/queue/scheduler // посмотреть текущий планировщик
grep "" /sys/block/*/queue/scheduler // все доступные планировщики
echo cfq > /sys/block/sda/queue/scheduler // задать планировщик для диска sda
free // читать буффер

hdparm // утилита для работы с прямым доступом к жёсткому диску
smartctl // для мониторинга состояния и работы жёсткого диска
fsck // может быть использована для проверки целостности файловой системы 
iostat // для мониторинга производительности устройств ввода-вывода.
lsof //  утилита для отображения открытых файлов и процессов

df //показывает информацию о доступном пространстве на файловых системах
du // позволяет оценить объём занимаемого файлами дискового пространства
sudo dd if=/dev/sda of=/home/dmitry/backup_mbr bs=512 count=1 //  выполняет копирование и преобразование файлов и устройств блоками

# Создадим файл нужного размера
$ sudo fallocate -l  1G /swapfile
$ sudo chmod 0600 /swapfile
# "Отформатируем" файл
$ sudo mkswap /swapfile
# Включаем подкачку
$ sudo swapon /swapfile
$ free

sudo dd if=/dev/zero of=~/radio1 bs=200M count=1 status=progress
sudo dd if=/dev/zero of=~/radio2 bs=200M count=1 status=progress
sudo losetup --find --show ~/radio1
sudo losetup --find --show ~/radio2
sudo mdadm --verbose --create /dev/md0 --level=1 --raid-devices=2 /dev/loop9 /dev/loop16
sudo mkfs.ext4 /dev/md0 
sudo mount /dev/md0 /mnt/raid/
sudo mdadm --detail --scan 
sudo mdadm --detail --scan  >> /etc/mdadm/mdadm.conf 
sudo update-initramfs -u 
sudo umount -r /mnt/raid 
sudo mdadm --stop /dev/md0 
sudo losetup -d /dev/loop9
sudo losetup -d /dev/loop16

strace ./process // посмотреть вызовы программы
strace -f ./process // системные вывзовы дочерних процессов
ps -U ${USER} -o pid,command,state //  Выведем процессы для нашего пользователя

cat /proc/loadavg // посмотреть загрузку процессора



sudo tcpdump -v -n -w curl.pcap // записать трафик
 curl -s https://practicum.yandex.ru > /dev/null // сделать запрос
dig practicum.yandex.ru  // резолв имени
host -t A  practicum.yandex.ru  // резолв IPv4
host -t AAAA practicum.yandex.ru // резолв IPv6
host -t CNAME practicum.yandex.ru // резолв перенаправления на другое доменное имя 
host -t PTR  87.250.250.5 // по IP получить доменное имя
cat /etc/nsswitch.conf // каонфигурация для того как искать адреса
/etc/hosts // файл с известными DNS
getent hosts practicum.yandex.ru // запрос в /etc/hosts
cat /etc/resolv.conf // конфигурация резолва
sudo ss -lp | grep 127.0.0.53 // посмотреть все порты которые слушают на хосте с названиями процессов
ps aux | grep 641 // найти процесс 641 
systemctl status systemd-resolved // статут сервиса
ip address show // показать состояние интерфесов
ip route show // таблица маршрутизации
arp -a // АРП таблица
ip neigh show // АРП таблица


/etc/netplan/*.yaml // файл настройки сети
$ netplan generate //сгенерировать конфигурацию серверной части из файлов netplan YAML
$ netplan apply // применить конфигурацию из файлов netplan YAML к работающей системе
$ netplan try // попробовать применить конфигурацию, при необходимости сделать откат изменений
ip a show eth0 // инфа про eth0


getent hosts //вывести список всех хостов из базы данных /etc/hosts
getent passwd // показать информацию о всех пользователях

curl -X POST -d "param1=value1&param2=value2" https://practicum.yandex.ru/ // отправить HTTP POST-запрос

curl -H "Authorization: <token>" https://practicum.yandex.ru/ //запрос с авторизацией через token
curl -H "Content-Type: application/json" -H "Authorization: <token>" -X POST -d {"test":"hello"} https://practicum.yandex.ru/api/test // отправить POST запрос с авторизацией и указанием типа тела запроса json

nc -u practicum.yandex.ru 1234 // установить UDP-соединение


ip a show // посмотреть все IP адреса, связанные с сетевыми интерфейсами
ip -br a show // та же информация, но в кратком виде
ip link show // посмотреть список сетевых интерфейсов
ss -ltn sport gt 8080 //список всех прослушивающие порты с номером порта = 8080 


file /boot/grub/i386-pc/boot.img // инфа про файл загрузчика GRUB
unmkinitramfs -v /boot/initrd.img-5.4.0-156-generic . // распаковать образ initrd 



Типы юнитов systemd

    service — отвечает за запуск сервисов (демонов) и поддерживает вызов интерпретаторов для исполнения пользовательских скриптов;
    target — группирует сервисы в группы, аналог runlevel;
    mount — занимается монтированием файловых систем;
    automount — автомонтирование файловых систем, используется при обращении к точке монтирования;
    swap — отвечает за подключение файла подкачки;
    timer — запускает модули по расписанию, аналог cron;
    socket — запуск модуля при подключении к сокету;
    slice — группировка других модулей в контейнер (дерево) cgroups;
    device — использует реакцию на подключение какого-либо устройства;
    path — запуск модуля по событию доступа по конкретному пути в файловой системе.



systemctl list-units --type service --all   — просмотр всех юнитов в системе
systemctl start name                        — запустить сервис
systemctl stop name                         — остановить сервис
systemctl restart name                      — перезапустить сервис
systemctl status name                       — посмотреть статус сервиса
systemctl reload name                       — перечитать конфигурацию
systemctl daemon-reload                     — перечитать конфигурацию для всех
systemctl try-restart name                  — перезапустить, если запущен
systemctl enable name                       — включить автозапуск сервиса
systemctl disable name                      — отключить автозапуск сервиса
systemctl list-unit-files --type service    — список установленных юнит-файлов сервисов


sudo journalctl --boot --unit nginx.service // записи с последней загрузки системы

echo "ibase=16; obase=2; $(stat -c %f test.txt | tr [:lower:] [:upper:])" | bc
1000000110110100 

umask // посмотреть текущую маску

id // Информацию об идентификаторах процесса и группы текущей оболочки


cat /etc/passwd | grep ubutnu // инфа про пользователей

echo $$ // PID текущей сессии

$ sudo getfattr -d -m '' -- /usr/bin/ping
# file: usr/bin/ping
security.capability=0sAQAAAgAgAAAAAAAAAAAAAAAAAAA= 
Параметр -d показывает расширенные атрибуты, -m '' показывает все атрибуты, а -- означает конец списка параметров и начало списка файлов. В выводе утилиты getfattr видно, что утилита ping имеет расширенный атрибут security.capability.

getcap /usr/bin/ping
/usr/bin/ping cap_net_raw=ep  // Для просмотра расшифровки атрибута
