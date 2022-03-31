**1.** Создаем VM в Google Cloud.

**2.** Ставим PostgresSQL 14  
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14*  
Ждем завершения установки.

**3.** Заходим в постгрес.  
*sudo -u postgres psql*

**4.** Задаем параметры кластера.  
*ALTER SYSTEM SET max_connections TO 40;  
ALTER SYSTEM SET shared_buffers TO '1GB'; 
ALTER SYSTEM SET effective_cache_size TO '3GB';  
ALTER SYSTEM SET maintenance_work_mem TO '512MB';  
ALTER SYSTEM SET checkpoint_completion_target TO 0.9;  
ALTER SYSTEM SET wal_buffers TO '16MB';  
ALTER SYSTEM SET default_statistics_target TO 500;  
ALTER SYSTEM SET random_page_cost TO 4;  
ALTER SYSTEM SET effective_io_concurrency TO 2;  
ALTER SYSTEM SET work_mem TO '6553kB';  
ALTER SYSTEM SET min_wal_size TO '4GB';  
ALTER SYSTEM SET max_wal_size TO '16GB';*  

**5.** Перезапускаем кластер, т.к. некоторые измененные параметры применяются при запуске.  
*sudo -u postgres pg_ctlcluster 14 main stop  
sudo -u postgres pg_ctlcluster 14 main start*

**6.** Заходим в PostgresSQL и устанавливаем пароль пользователю postgres.  
*sudo -u postgres psql  
alter user postgres password '123';*

**7.** Инициализируем бенчмарк.  
*pgbench -i postgres -U postgres -h 127.0.0.1*

**8.** Запускаем бенчмарк и ждем завершения.  
*pgbench -c8 -P 60 -T 3600 -U postgres -h 127.0.0.1 postgres*  
Среднее значение tps в конечной части работы около 370, а за весь час 420

**9.** Перенастроем autovacuum.  
*ALTER SYSTEM SET autovacuum_max_workers TO 6;* - Поднимем количество воркеров до 6  
*ALTER SYSTEM SET autovacuum_vacuum_scale_factor TO 0.5;* - Уменьшим частоту срабатывания  
\q

**10.** Перезапускаем PostgresSQL.  
*sudo -u postgres pg_ctlcluster 14 main stop  
sudo -u postgres pg_ctlcluster 14 main start*

**11.** Запускаем бенчмарк и ждем завершения.  
Получили средний tps 296 [scrin](https://github.com/monstermurm/postgres2022/blob/main/05-%D0%94%D0%97/Gafic.png)

Перепробовал несколько вариантов, но конечный дал очень малый разброс.
Однако каждый следующий тест давал потерю производительности к предыдущему, даже если параметры менялись не сильно, это пока не могу объяснить.
