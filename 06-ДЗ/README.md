**1.** Создаем VM в Google Cloud.

**2.** Ставим PostgresSQL 14  
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14*  
Ждем завершения установки.

**3.** Заходим в постгрес.  
*sudo -u postgres psql*

**4.** Задаем параметры кластера.  
*ALTER SYSTEM SET checkpoint_timeout TO '30s';  
ALTER SYSTEM SET log_checkpoints = on;*

**5.** Получаем точку журнала.  
*SELECT pg_current_wal_insert_lsn();  
0/3243C70*

**6.** Запускаем нагрузочное тестирование на 10 мин.  
*pgbench -c8 -T 600 -U postgres -h 127.0.0.1 postgres*

**7.** Получаем точку журнала и считаем получаем размер между точками.  
*SELECT pg_current_wal_insert_lsn();  
0/1B6B9520  
SELECT pg_size_pretty('0/1B6B9520'::pg_lsn - '0/3243C70'::pg_lsn);  
388 MB  
одна контрольная точка = 388 / 20 = 19.4 Mb*

**8.** Проверяем работу точек по расписанию.  
*2022-04-04 13:38:39.957 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:39:09.040 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:39:39.064 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:40:09.131 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:40:39.145 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:41:09.097 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:41:39.040 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:42:09.146 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:42:39.060 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:43:09.103 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:43:39.075 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:44:09.179 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:44:39.032 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:45:09.136 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:45:39.135 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:46:09.042 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:46:39.080 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:47:09.076 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:47:39.083 UTC [7534] LOG:  checkpoint starting: time  
2022-04-04 13:48:09.077 UTC [7534] LOG:  checkpoint starting: time*  
При нагрузочном тестировании точки в целом создавались какждые 30 сек с небольшими отклонениями по мс, после завершения тестирования, время создания увеличилось.  
*2022-04-04 13:49:09.075 UTC [7534] LOG:  checkpoint starting: time*  
т.к. данные перестали изменяться

**9.** Проверяем скорость работы в синхронном режиме.  
*ALTER SYSTEM SET synchronous_commit TO on;  
pgbench -c8 -T 600 -U postgres -h 127.0.0.1 postgres  
tps = 373.009643*

**10.** Проверяем скорость работы в асинхронном режиме.  
*ALTER SYSTEM SET synchronous_commit TO off;  
pgbench -c8 -T 600 -U postgres -h 127.0.0.1 postgres  
tps = 1026.991300*  
Скорость сильно возрасла, т.к. постгрес не ожидал подтверждения записи контрольной точки на диск.

**11.** Создаем кластер с контролем четности таблиц.  
*sudo pg_createcluster 14 test -- --data-checksums*

**12.** Создадим таблицу и заполним ее.  
*create table test (t int);  
insert into test values (1),(2),(3);*

**13.** Находим расположение таблицы, останавливаем сервер и меняем пару байт в таблице.
*SELECT pg_relation_filepath('test');  
sudo -u postgres pg_ctlcluster 14 test stop  
sudo dd if=/dev/zero of=/var/lib/postgresql/14/test/base/13760/16384 oflag=dsync conv=notrunc bs=1 count=5*  

**14.** Стартуем кластер и пытаемся прочитать из таблицы.  
*select * from test;  
WARNING:  page verification failed, calculated checksum 12216 but expected 57802  
 t  
  
 1  
 2  
 3  
(3 rows)*  
в теории тут долюна быть ошибка, которую можно обойти через параметр *ignore_checksum_failure*  
но у меня просто предупреждение и результат из таблицы  
при этом проверка режима чек сум показывает  
*SHOW data_checksums;  
 data_checksums  
    
 on  
(1 row)*  
Что, контрольные суммы включены. Может я попал в пустую область страницы, поэтому отделался только предупреждением?
