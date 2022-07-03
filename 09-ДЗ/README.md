**1.** Создаем 4 VM с именами server1, server2, server3 и server4.  

**2.** На server1, server2 и server3 создаем базу.  
*create database db*

**3.** На server1, server2 и server3 создаем таблицы.  
*create table table1(id int, txt text);  
create table table2(id int, txt text);*

**4.** На server1, server2 и server3 правим файлы pg_hba.conf и postgresql.conf для доступа по сети.  
Так же для работы логической репликации нам надо изменить wal_level  
*ALTER SYSTEM SET wal_level = logical;*  
не забываем рестартануть службу  
*sudo -u postgres pg_ctlcluster 14 main stop  
sudo -u postgres pg_ctlcluster 14 main start*

**5.** Создаем публикации:  
на server1  
*create publication pb_test1 for table1 test1;*  
на server2  
*create publication pb_test2 for table2 test2;*  

**6.** Создаем пользователя для подключения на server1 и server2.    
*CREATE ROLE test WITH LOGIN PASSWORD '123' IN ROLE postgres SUPERUSER;*

**7.** Создаем подписки:  
на server1  
*create subscription sb_test2 CONNECTION 'host=192.168.0.24 port=5432 user=test password=123 dbname=db' publication pb_test2 with (copy_data = true);*  
на server2  
*create subscription sb_test1 CONNECTION 'host=192.168.0.57 port=5432 user=test password=123 dbname=db' publication pb_test1 with (copy_data = true);*  
на server3  
*create subscription sb_test2 CONNECTION 'host=192.168.0.24 port=5432 user=test password=123 dbname=db' publication pb_test2 with (copy_data = true);*  
*create subscription sb_test1 CONNECTION 'host=192.168.0.57 port=5432 user=test password=123 dbname=db' publication pb_test1 with (copy_data = true);*

**8.** Останавливаем службу на server4, очищаем содержимое /etc/postgresql/14/main/ и запускаем бэкап с server3  
*sudo -u postgres pg_ctlcluster 14 main stop  
sudo -u postgres pg_basebackup -p 5432 -h 192.168.0.63 -R -D /etc/postgresql/14/main*

в результате  
1) изменения в таблице test1 на server1 будут отображаться на server2  
2) изменения в таблице test2 на server2 будут отображаться на server1  
3) все изменения таблиц test1 и test2 на серверах server1 и server2 будут отображаться на server3  
4) все это дело с server3 будет полностью реплицироваться на server4  
