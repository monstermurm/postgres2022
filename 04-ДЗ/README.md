**Домашнее задание 4.**

**1.** Создаем VM.

**2.** Устанавливаем PostgresSQL и заходим в него.  
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14  
sudo -u postgres psql*

**3.** Создаем БД testdb и подключаемся к ней.  
*CREATE DATABASE testdb;  
\c testdb*

**4.** Создаем схему testnm.  
*CREATE SCHEMA testnm;*

**5** Создаем таблицу t1.  
*CREATE TABLE t1(c1 integer);*

**6/** Добавляем запись в таблицу t1.  
*INSERT INTO t1 values(1);*

**7.** Создаем роль readonly.  
*CREATE role readonly;*

**8.** Даем доступ роли readonly на подключение к базе testdb.  
*grant connect on DATABASE testdb TO readonly;*

**9.** Даем доступ к схеме testnm роли readonly.  
*grant usage on SCHEMA testnm to readonly;*

**10.** Даем доступ на селект всех таблиц из схемы testnm для роли readonly.  
*grant SELECT on all TABLEs in SCHEMA testnm TO readonly;*

**11.** Создаем пользователя testread с паролем test123.  
*CREATE USER testread with password 'test123';*

**12.** Присваиваем роль readonly пользователю testread.  
*grant readonly TO testread;*

**13.** Подключаемся к testdb от пользователя testread.  
*\q  
sudo psql -h 127.0.0.1 -U testread -d testdb -W*
поскоьку у нас пользователь testread существует только в postgresql, то подключаемся с полным указанием параметров соединения

**14.** Пробуем прочитать данные из таблицы t1.  
*SELECT * FROM t1;*  
Полчаем ошибку, *ERROR:  permission denied for table t1* т.к. таблица у нас по дефолту создалась в схеме public, а прав роли readonly на эту схему не давали

**15.** Подключаемся к таблице testdb от пользователя postgres.  
*\q  
sudo -u postgres psql  
\c testdb*

**16.** Удаляем таблицу t1.  
*drop TABLE t1;*

**17.** Создаем таблицу t1 в схеме testnm и добавляем в нее данные.  
*CREATE TABLE testnm.t1(c1 integer);  
INSERT INTO testnm.t1 values(1);*

**18.** Подключаемся к базе testdb от пользователя testread.  
*\q  
sudo psql -h 127.0.0.1 -U testread -d testdb -W*

**19.** Пробуем читать из таблицы t1 по схеме testnm.  
*select * from testnm.t1;*  
Получаем ошибку *ERROR:  permission denied for table t1*, связано это с тем, что таблица была пересоздана и на нее не выданы права пользователю testread на select.

**20.** Чтобы пользователь имел всегда доступ на чтение таблицы из схемы, мы можем задать привелегию по умолчанию.  
*ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;*

**21.** Пробуем читать из таблицы t1 по схеме testnm.  
*select * from testnm.t1;*  
Получаем ошибку *ERROR:  permission denied for table t1*, связано это с тем, что выданные дефолтовые права будут действовать на новые таблицы, а не на уже существующие.   
Поэтому нам надо выдать права на чтение повторно.  
*grant SELECT on all TABLEs in SCHEMA testnm TO readonly;*

**22.** Читаем данные из таблицы t1.  
*select * from testnm.t1;*  
Все получилось.

**23.** Создаем таблицу t2 и добавляем в нее запись от пользователя testread.  
*create table t2(c1 integer);  
insert into t2 values (2);*  
Не смотря на то, что мы права не дали, мы можем это делать из-за схемы по умолчанию public с ролью на создание public, которая присваивается по умолчанию для всех новых пользователей.

**24.** Чтобы этого избежать, надо запретить роли public создавать новые объекты.  
*revoke CREATE on SCHEMA public FROM public;*  
И отключить права от роли public в нашей БД.  
*revoke all on DATABASE testdb FROM public;*

**25.** После этого если мы опять попробуем создать таблицу  
*create table t3(c1 integer);*  
то мы получим ошибку прав доступа: *ERROR:  permission denied for schema public*  
если попробуем вставить данные в таблицу t2  
*insert into t2 values (2);*  
то у нас все получиться, это потому что права на вставку таблицы t2 мы не забрали.
