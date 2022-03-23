**Домашнее задание 2.**

**1.** Создаем VM в Google Cloud.

**2.** Ставим PostgresSQL 14
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14*
Ждем завершения установки.

**3.** Проверяем, что сервер работает
*pg_lsclusters*
Видим наш запущенный кластер.

**4.** Заходим в постгрес
*sudo -u postgres psql*

**5.** Создаем таблицу
*create table test(a int, c text);*
Добавляем запись в таблицу
*insert into test values(1, 'test');*
Выходим
*\q*

**6.** Останавливаем кластер.
*sudo -u postgres pg_ctlcluster 14 main stop*

**7.** Создаем диск

**8.** Аттачим диск к VM

**9.** Инициализируем диск и монтируем согласно инструкции
*sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mklabel msdos
sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L datapartition /dev/sdb1
sudo mkdir -p /mnt/data
sudo mount -o defaults /dev/sdb1 /mnt/data*

**10.** Делаем пользователя postgres владельцем /mnt/data
*sudo chown -R postgres:postgres /mnt/data/*

**11.** Переносим содержимое /var/lib/postgres/14 в /mnt/data
*sudo mv /var/lib/postgresql/14 /mnt/data*

**12.** Пытаемся запустить PostgresSQL
*sudo -u postgres pg_ctlcluster 14 main start*
Получаем ошибку, причина в том, что мы базы перенесли в другое место.

**13.** Открываем файл postgresql.conf
*sudo nano /etc/postgresql/14/main/postgresql.conf*
и меняем путь данных с /var/lib/postgresql/14/main на /mnt/data/14/main

**14.** Запускаем класетр
*sudo -u postgres pg_ctlcluster 14 main start*
Проверяем, что работает.
*pg_lsclusters*

**15.** Заходим в PostgresSQL
*sudo -u postgres psql*
Проверяем содержимое созданной таблицы.
*select * from test;*
Таблица и наши данные на месте.

**16.** Отмонтировали диск.

**17.** Создаем новую VM.

**18.** Аттачим к ней диск.

**19.** Ставим на новую VM PostgresSQL 14

**20.** Монтируем диск.
*sudo mkdir -p /mnt/data
sudo mount -o defaults /dev/sdb1 /mnt/data*
**21.** Делаем пользователя postgres владельцем /mnt/data
*sudo chown -R postgres:postgres /mnt/data/*

**22.** Останавливаем PostgresSQL
*sudo -u postgres pg_ctlcluster 14 main stop*

**23.** Открываем файл postgresql.conf
*sudo nano /etc/postgresql/14/main/postgresql.conf*
и меняем путь данных с /var/lib/postgresql/14/main на /mnt/data/14/main

**25.** Запускаем класетр
*sudo -u postgres pg_ctlcluster 14 main start*
получаем предупреждение, но если проверить работу кластера
*pg_lsclusters*
то он работает

**26.** Заходим в PostgresSQL.
*sudo -u postgres psql*

**27.** Проверяем нашу тестовую таблицу.
*select * from test;*
И видим, что таблица и данные на месте.
Таким образом мы перенесли наши базы с однй VM на другую.
Недостатком пока этого является то, что PostgresSQL не хочет работать как сервис, надо что-то с pid файлом сделать, тут я пока не знаю.
