# Сервер на базе Raspberry PI 3 от Armbian OpenMediaValut


Raspberry Pi 3 — полноценный бесшумный компьютер размером с банковскую карту. 4 Оснащён 64-х битным четырёхъядерным процессором ARM Cortex-A53 с тактовой частотой 1,2 ГГц на однокристальном чипе Broadcom BCM2837. 24


[Официальное описание Raspberry PI 3](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/)

Поддерживают разные OS - Android, Ubuntu, Debian, Raspbian, zeroshell.


Ни скажу ничего про Raspbian, zeroshell и Android, так как не вижу смысла ставить их на Zero, а вот linux системы староваты. Ubuntu server версии 14.

На просторах сети набрел на сборки от [Armbian OpenMediaValut](https://dl.armbian.com/rpi4b/Bookworm_current_minimal-omv) на базе 12 версии bookworm. Загрузка в разы быстрее, нет конфликтов в пакетах, более новые версии программ, например php7. Свежие сборки, давностью не более месяца.

# Задача

Использовать Raspberry PI 3 как сервер для хранения файлов и доступа к ним по сети. Также как web сервер с поддержкой php и mysql , ftp , dlna , openmediavalut , docker , filebrowser.

# Что в наличии

Raspberry PI - 1 штука
Плата расширения - 1 штука
Адаптер USB 1 Ватт - 1 штука
Радиатор алюминиевый - 1 штука
Карта micro SD, Class 10, 8 Gb - 1 штука

Совет по выбору SD карты - нужно выбирать карту с максимальной скоростью записи и чтения. Обычно это Class 10 с маркировкой U3 где скорость записи выше 30 мб в секунду, хотя иногда это просто Class 10 без маркировок, тут надо внимательно смотреть описание.

# Программы

[Armbian OpenMediaValut](https://dl.armbian.com/rpi4b/Bookworm_current_minimal-omv)

Программа для чтения и записи образов (.img) на флеш карту.
[etcher.io](https://etcher.io/)
[Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)

Программа для подключения по ssh.
[putty](https://www.putty.org/)

Программа для сканирования сети для определения IP адреса мини-пк при первом подключении.
[Advanced IP Scanner](http://www.advanced-ip-scanner.com/ru/)

# Установка ArmBian Raspberry PI 3

Пишем на флешку образ ubuntu server.

Вставляем флешку в мини пк. Подключаем питание и сеть.

IP можно посмотреть через роутер, либо использовать сканер сети, например вот этот [Advanced IP Scanner](http://www.advanced-ip-scanner.com/ru/)

Запускаем putty и подключаемся к IP ардуины.

![putty](/images/2024-02-07_20-05-40.png)

Появится окно терминала с запросом логина и пароля.

login: root

password: 1234

Далее вас попросят ввести пароль для root. Затем задать новые пароль, создать пользователя, задать для него пароль, и ввести прочую регистрационную информацию.

![first login](/images/2024-02-07_20-12-36.png)

После регистрации для входа лучше использовать заданное вами имя пользователя вместо root.

# armbian-config

Для настройки системы можно использовать программу armbian-config... Через нее можно и сеть настроить, и программы поставить, но это не наш путь...

# Обновление

```
sudo apt-get update
sudo apt-get upgrade
```

после успешного выполнения

```
sudo reboot
```

# Установка фиксированного IP

Для настройки IP используем утилиты nmtui, nmcli либо armbian-config. Настройка через /etc/network/interfaces как я понял deprecated. По другой информации все разбили на файлы... Итого: ничего не понятно... Бесит.

## nmcli

Получаем список соединений

```
nmcli connection show
```

В списке находим название соединения и получаем подробную информацию

```
nmcli connection show 'Wired connection 1'
```

Модифицируем нужные нам параметры (имена параметров смотрим в списке полученном ранее)

```
sudo nmcli connection modify 'Wired connection 1' ipv4.address 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 192.168.1.1 +ipv4.dns 8.8.8.8
```

Перезагрузка

```
sudo reboot
```

# Установка timezone

```
sudo dpkg-reconfigure tzdata
```

# Установка Apache, PHP и MySQL

```
sudo apt-get install apache2 php7.0 php7.0-gd php7.0-json php7.0-mysql php7.0-xml php7.0-bz2 libapache2-mod-php7.0 php7.0-mbstring mysql-client mysql-server
```

Включаем mysqli

```
sudo phpenmod mysqli
```

Если все прошло успешно, то по адресу http://192.168.1.100/ будет страница с информацией об apache. Находится она по адресу /var/www/html/index.html. На странице много полезной информации, но нам она не нужна, удаляем ее.

```
sudo rm -f /var/www/html/index.html
```

Или переименовываем

```
sudo mv /var/www/html/index.html /var/www/html/apache_index.html
```

По умолчанию document_root сервера /var/www/html. Папка защищена от записи, лежит в системном каталоге, сменить ее жуткая морока... По этому создадим папку в домашнем каталоге и зададим на нее постоянную ссылку. Папку пользователя использовать не будем, так как в ней хранится системный мусор которые не стоит показывать людям =) (история bash, mysql и т.п.) Создадим папку html которую смонтируем в виде постоянной ссылки под именем mrgobus, тогда папка html на сервере будет видна как http://192.168.1.100/.

Подобная ссылка называется [symlink](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%BC%D0%B2%D0%BE%D0%BB%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F_%D1%81%D1%81%D1%8B%D0%BB%D0%BA%D0%B0)

За одно, сменим права доступа для папки html на полный доступ (777). Надо это для того, чтобы php мог сохранять файлы. Возможно, можно сделать это как то иначе, но я не знаю как. Думаю для домашнего сервера это не особо критично.

```
mkdir html
sudo ln -s ~/html /var/www/html/mrgobus
chmod 777 html
```

# Установка Samba

[Samba](https://ru.wikipedia.org/wiki/Samba) — пакет программ, которые позволяют обращаться к сетевым дискам и принтерам на различных операционных системах по протоколу SMB/CIFS.

Самое главное - Samba позволят шарить папки по сети Windows.

Ставим и создаем пользователя

```
sudo apt-get install samba
sudo smbpasswd -a serverlb
```

Правим файл настроек

```
sudo nano /etc/samba/smb.conf
```

В конец добавляем что-то вроде, чтобы расшарить папку.

```
[mrgobus]
path = /home/serverlb
valid users = 
read only = No
guest ok = Yes
```

Перезапуск самбы

```
sudo service smbd restart
```

Все, самба работает. Можно заходить через сетевое окружение Шиндовс =)

# Установка FTP

```
sudo apt install vsftpd
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```

Правим файл настройки

```
sudo nano /etc/vsftpd.conf
```

Заменяем строки

```
# Связываем доступ с локальными пользователями
local_enable=YES

# Включаем запись
write_enable=YES

# Запираем пользователя в корневом каталоге
chroot_local_user=YES

# Маска создания файлов
# вычисляется по формуле: маска = 777 - <требуемая маска>
local_umask=022
```

Добавляем строку

```
# Разрешаем пользователям запись в корневой каталог
allow_writeable_chroot=YES
```

Перезапускаем vsftpd

```
sudo service vsftpd restart
```

Фтп готово, проверяем любым FTP клиентом

# Удаленный доступ к MySQL

```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Устанавливаем.

```
bind-address = 0.0.0.0
```

Перезапускаем mysql.

```
sudo service mysql restart
```

Запускаем клиент.

```
mysql –u root -p
```

Выполняем.

В место слова 'password' нужно указать ваш пароль.

```
GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Выходим из клиента.

```
exit;
```

Проверить результат можно любым клиентом MySQL подключившись удаленно (например [MySQL Workbench](https://www.mysql.com/products/workbench/))

# node.js

Собираем классическим образом из исходников, так как в репазиториях версия 4.x когда на данный момент актуальна 9.x. Node.js крупный проект по этому собирается долго, где-то часов пять - восемь. Для того чтобы не держать SSH соединение во время сборки пользуемся утилитой screen - виртуальный экран консоли к которому можно подключиться при следующем соединении.

```
screen
```

Запускаем сборку

```
wget http://nodejs.org/dist/node-latest.tar.gz
tar -xzf node-latest.tar.gz
cd [node folder]
./configure
make
sudo make install
```

После запуска make начнется долгая часть сборки и если вы запустили сборку в screen то можно выключить putty (например на ночь), и при следующем заходе вернуться к выполняемой задаче что-бы проследить ход выполнения либо продолжить сборку. Orange PI выключать не надо, она собирает =)

```
screen -r
```

По завершении закрываем экран

```
exit
```
# Свап файл

Нода требовательна к памяти, и если ее нехватит то выдаст killed. Лечится это подключением свап файла, который отключен по умолчанию. Для свапа используется какаято zram, но ее катастрафически мало (порядка 60 мб).

Свап файл включаем так.

```
sudo fallocate -l 5G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show
sudo cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl vm.vfs_cache_pressure=50
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
```

Отключение /swapfile (может пригодиться для изменения размера свап файла)

```
sudo swapoff -v /swapfile

```

# Настройка OpenMediaValut

[Настройка](https://pcminipro.ru/os/linux-armbian/ustanovka-open-media-vault-omv-nas-server-v-armbian-debian/)

# Итого

В теории у нас получился рабочий сервер с HTTP, PHP7, MySQL, FTP, Samba. Для проверки я установил phpMyAdmin который работает, создает базы и выполняет свои функции. При первом запуске ругался на отсутствие mysqli, решение добавил в раздел установки php (sudo phpenmod mysqli).

Установил Wordpress. Файл wp-config.php пришлось создать вручную. MySQL подтормаживает при записи, что сказывается на работе админки, но жить можно. Думаю виной тут SD карта с сомнительной скоростью записи.

Теоретически есть возможность установить систему и грузиться с жесткого дика. Но пока у меня нет адаптера и проверить это я не могу.

Видео в формате 1080 заливается и читается. Просмотр без тормозов.

# P.S.

Не забываем бекапить карту с помощью Win32 Disk Imager чтобы не повторять все снова если что-то пойдет не так =)

