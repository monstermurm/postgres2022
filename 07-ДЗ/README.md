**1.** Создаем VM в Google Cloud.

**2.** Ставим PostgresSQL 14  
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14*  
Ждем завершения установки.

**3.** Заходим в постгрес.  
*sudo -u postgres psql*

**4.** Задаем параметры кластера.  
*ALTER SYSTEM SET log_lock_waits TO on;  
ALTER SYSTEM SET deadlock_timeout TO '200ms';*

**5.** Создаем таблицу и заполняем ее.  
*create table test(id int);  
insert into test values (1), (2), (3);*

**6.** Открываем 3 сеанса и пытаемя обновить одну и ту же запись.  
*update test set id = 1 where id = 1;  
update test set id = 20 where id = 1;  
update test set id = 30 where id = 1;*

**7.** Смотрим возникшие блокировки.  
*SELECT pid, locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted FROM pg_locks;*

 <pid  |   locktype    | relation | virtxid |   xid   |       mode       | granted
------+---------------+----------+---------+---------+------------------+---------
 1146 | relation      | pg_locks |         |         | AccessShareLock  | t
 1146 | virtualxid    |          | 6/21    |         | ExclusiveLock    | t
 1016 | relation      | test     |         |         | RowExclusiveLock | t
 1016 | virtualxid    |          | 5/15    |         | ExclusiveLock    | t
  939 | relation      | test     |         |         | RowExclusiveLock | t
  939 | virtualxid    |          | 4/31    |         | ExclusiveLock    | t
  838 | relation      | test     |         |         | RowExclusiveLock | t
  838 | virtualxid    |          | 3/23    |         | ExclusiveLock    | t
  838 | transactionid |          |         | 1584469 | ExclusiveLock    | t
 1016 | transactionid |          |         | 1584471 | ExclusiveLock    | t
  939 | transactionid |          |         | 1584470 | ExclusiveLock    | t
 1016 | tuple         | test     |         |         | ExclusiveLock    | f
  939 | transactionid |          |         | 1584469 | ShareLock        | f
  939 | tuple         | test     |         |         | ExclusiveLock    | t>

3 соединения наложили экслюзивную блокировку на строку RowExclusiveLock.  
Соединение 838 заблокировало строку, соединение 939 создало очередь ожидания tuple, а соединение 1016 встало в эту очередь.  
Если мы запустим еще одно апдейт, то новое соединение так же встанет в очередь и будет ожидать tuple.

**8.** Смотрим лог.

<2022-04-11 16:23:31.222 UTC [838] postgres@postgres STATEMENT:  update test set id = 10 where id = 1;
2022-04-11 16:23:31.422 UTC [838] postgres@postgres LOG:  process 838 still waiting for ShareLock on transaction 1584467 after 200.121 ms
2022-04-11 16:23:31.422 UTC [838] postgres@postgres DETAIL:  Process holding the lock: 1016. Wait queue: 838.
2022-04-11 16:23:31.422 UTC [838] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-11 16:23:31.422 UTC [838] postgres@postgres STATEMENT:  update test set id = 10 where id = 1;
2022-04-11 16:23:58.300 UTC [838] postgres@postgres LOG:  process 838 acquired ShareLock on transaction 1584467 after 27077.714 ms
2022-04-11 16:23:58.300 UTC [838] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-11 16:23:58.300 UTC [838] postgres@postgres STATEMENT:  update test set id = 10 where id = 1;
2022-04-11 16:24:21.504 UTC [939] postgres@postgres LOG:  process 939 still waiting for ShareLock on transaction 1584469 after 200.133 ms
2022-04-11 16:24:21.504 UTC [939] postgres@postgres DETAIL:  Process holding the lock: 838. Wait queue: 939.
2022-04-11 16:24:21.504 UTC [939] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-11 16:24:21.504 UTC [939] postgres@postgres STATEMENT:  update test set id = 20 where id = 1;
2022-04-11 16:24:28.565 UTC [1016] postgres@postgres LOG:  process 1016 still waiting for ExclusiveLock on tuple (0,5) of relation 16411 of database 13760 after 200.148 ms
2022-04-11 16:24:28.565 UTC [1016] postgres@postgres DETAIL:  Process holding the lock: 939. Wait queue: 1016.
2022-04-11 16:24:28.565 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 1;>

Здесь мы видим порядок наложенных блокировок, а так же предупреждения о длительности больше 200ms.

**9.** Вызовем взаимоблокировку из этой ситуации.

<2022-04-11 16:35:40.879 UTC [1016] postgres@postgres ERROR:  canceling statement due to user request
2022-04-11 16:35:40.879 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 1;
2022-04-11 16:36:08.477 UTC [1016] postgres@postgres ERROR:  current transaction is aborted, commands ignored until end of transaction block
2022-04-11 16:36:08.477 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 3;
2022-04-11 16:36:35.388 UTC [1016] postgres@postgres LOG:  process 1016 still waiting for ExclusiveLock on tuple (0,5) of relation 16411 of database 13760 after 200.135 ms
2022-04-11 16:36:35.388 UTC [1016] postgres@postgres DETAIL:  Process holding the lock: 939. Wait queue: 1016.
2022-04-11 16:36:35.388 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 1;
2022-04-11 16:38:04.474 UTC [838] postgres@postgres LOG:  process 838 detected deadlock while waiting for ShareLock on transaction 1584472 after 200.190 ms
2022-04-11 16:38:04.474 UTC [838] postgres@postgres DETAIL:  Process holding the lock: 1016. Wait queue: .
2022-04-11 16:38:04.474 UTC [838] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "test"
2022-04-11 16:38:04.474 UTC [838] postgres@postgres STATEMENT:  update test set id = 30 where id = 3;
2022-04-11 16:38:04.474 UTC [838] postgres@postgres ERROR:  deadlock detected
2022-04-11 16:38:04.474 UTC [838] postgres@postgres DETAIL:  Process 838 waits for ShareLock on transaction 1584472; blocked by process 1016.
        Process 1016 waits for ExclusiveLock on tuple (0,5) of relation 16411 of database 13760; blocked by process 939.
        Process 939 waits for ShareLock on transaction 1584469; blocked by process 838.
        Process 838: update test set id = 30 where id = 3;
        Process 1016: update test set id = 30 where id = 1;
        Process 939: update test set id = 20 where id = 1;
2022-04-11 16:38:04.474 UTC [838] postgres@postgres HINT:  See server log for query details.
2022-04-11 16:38:04.474 UTC [838] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "test"
2022-04-11 16:38:04.474 UTC [838] postgres@postgres STATEMENT:  update test set id = 30 where id = 3;
2022-04-11 16:38:04.474 UTC [939] postgres@postgres LOG:  process 939 acquired ShareLock on transaction 1584469 after 823170.325 ms
2022-04-11 16:38:04.474 UTC [939] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-11 16:38:04.474 UTC [939] postgres@postgres STATEMENT:  update test set id = 20 where id = 1;
2022-04-11 16:38:04.475 UTC [1016] postgres@postgres LOG:  process 1016 acquired ExclusiveLock on tuple (0,5) of relation 16411 of database 13760 after 89286.848 ms
2022-04-11 16:38:04.475 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 1;
2022-04-11 16:38:04.675 UTC [1016] postgres@postgres LOG:  process 1016 still waiting for ShareLock on transaction 1584470 after 200.181 ms
2022-04-11 16:38:04.675 UTC [1016] postgres@postgres DETAIL:  Process holding the lock: 939. Wait queue: 1016.
2022-04-11 16:38:04.675 UTC [1016] postgres@postgres CONTEXT:  while updating tuple (0,5) in relation "test"
2022-04-11 16:38:04.675 UTC [1016] postgres@postgres STATEMENT:  update test set id = 30 where id = 1;>

Порядок возникновения взаимоблокировке виден по времени поступления команд и какой сеанс с каким вступил в конфликт.  
838 Заблокировал запись с id 1  
1016 заблокировал запись с id 3  
после чего 1016 встал ожидая записи с id 1, а 838 встал ожидая запись с id 3.  

**10.** Две транзакции на update по одной и той же таблице могут заблокировать друг друга.

Заполним тестовую таблицу данными и создадим индекс.  
*insert into test (id) (SELECT generate_series(1, 10000000));   
CREATE INDEX ON test(id DESC);*  
Запрещаем последовательное сканирование.  
*SET enable_seqscan = off;*

В первой транзакции запускаем обновление id.  
*UPDATE test SET id = id + 1;*

Во второй транзакции запускаем обновление id.  
*UPDATE test SET id = id + 10000000;*

Наша задача, чтобы одни и те же строки попали на апдейт в разные транзакции в разном порядке, тогда будет deadlock.
