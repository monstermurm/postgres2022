**1.** Устанавливаем постгрес.

**2.** Устанавливаем pgbench.  
*pgbench -i postgres -U postgres -h 127.0.0.1*

**3.** Запускаем pgbench на дефолтных настройках.  
*pgbench -c10 -P 60 -T 600 -U postgres -h 127.0.0.1 postgres*

    progress: 60.0 s, 589.2 tps, lat 16.915 ms stddev 13.752
    progress: 120.0 s, 705.1 tps, lat 14.148 ms stddev 11.174
    progress: 180.0 s, 744.0 tps, lat 13.408 ms stddev 9.760
    progress: 240.0 s, 688.9 tps, lat 14.483 ms stddev 11.311
    progress: 300.0 s, 712.9 tps, lat 13.998 ms stddev 10.271
    progress: 360.0 s, 670.2 tps, lat 14.889 ms stddev 11.600
    progress: 420.0 s, 750.4 tps, lat 13.292 ms stddev 9.818
    progress: 480.0 s, 651.4 tps, lat 15.318 ms stddev 12.887
    progress: 540.0 s, 731.7 tps, lat 13.631 ms stddev 10.159
    progress: 600.0 s, 762.0 tps, lat 13.090 ms stddev 9.603
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 10
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 420364
    latency average = 14.239 ms
    latency stddev = 11.066 ms
    initial connection time = 75.674 ms
    tps = 700.665602 (without initial connection time)

**4.** Запускаем утилиту PG tune  
Сервер 2 процессора  
ОС Linux Ubuntu 20.4 TSL  
RAM 4 Gb  
HDD обычный 15 Gb  
DB Type OLTP  

*max_connections = 20  
shared_buffers = 1GB  
effective_cache_size = 3GB  
maintenance_work_mem = 256MB  
checkpoint_completion_target = 0.9  
wal_buffers = 16MB  
default_statistics_target = 100  
random_page_cost = 4  
effective_io_concurrency = 2  
work_mem = 52428kB  
min_wal_size = 2GB  
max_wal_size = 8GB  
max_worker_processes = 2  
max_parallel_workers_per_gather = 1  
max_parallel_workers = 2  
max_parallel_maintenance_workers = 1*

**5.** Запускаем pgbench на дефолтных настройках.  
*pgbench -c10 -P 60 -T 600 -U postgres -h 127.0.0.1 postgres*

    progress: 60.0 s, 658.1 tps, lat 15.136 ms stddev 12.343
    progress: 120.0 s, 662.7 tps, lat 15.057 ms stddev 20.390
    progress: 180.0 s, 763.7 tps, lat 13.062 ms stddev 9.229
    progress: 240.0 s, 724.5 tps, lat 13.769 ms stddev 10.610
    progress: 300.0 s, 741.2 tps, lat 13.456 ms stddev 10.236
    progress: 360.0 s, 752.0 tps, lat 13.264 ms stddev 10.127
    progress: 420.0 s, 732.6 tps, lat 13.617 ms stddev 10.171
    progress: 480.0 s, 624.4 tps, lat 15.979 ms stddev 13.697
    progress: 540.0 s, 750.5 tps, lat 13.293 ms stddev 9.797
    progress: 600.0 s, 747.9 tps, lat 13.337 ms stddev 9.783
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 10
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 429461
    latency average = 13.936 ms
    latency stddev = 11.937 ms
    initial connection time = 78.919 ms
    tps = 715.810229 (without initial connection time)

**6.** Как видим прирост совсем не большой 2%.  
Предположительно связано это с низкой скоростью диска, т.к. параметры памяти от дефолтных мы подняли довольно сильно, проверим снизив на него нагрузку.

*alter system SET synchronous_commit to off;* - отключим синхронизацию сброса wall, тут мы рискуем потерять небольшую часть данных  
*alter system SET fsync to off;* - отключим синхронизацию сброса данных кэша, тут же база может быть повреждена серьезно

    progress: 60.0 s, 2741.8 tps, lat 3.592 ms stddev 1.744
    progress: 120.0 s, 2728.9 tps, lat 3.613 ms stddev 1.747
    progress: 180.0 s, 2584.1 tps, lat 3.819 ms stddev 1.818
    progress: 240.0 s, 3017.7 tps, lat 3.268 ms stddev 1.604
    progress: 300.0 s, 3101.6 tps, lat 3.179 ms stddev 1.570
    progress: 360.0 s, 3137.0 tps, lat 3.143 ms stddev 1.550
    progress: 420.0 s, 3131.4 tps, lat 3.148 ms stddev 1.559
    progress: 480.0 s, 3144.6 tps, lat 3.135 ms stddev 1.549
    progress: 540.0 s, 3132.6 tps, lat 3.147 ms stddev 1.555
    progress: 600.0 s, 3166.8 tps, lat 3.113 ms stddev 1.516
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 10
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 1793210
    latency average = 3.299 ms
    latency stddev = 1.635 ms
    initial connection time = 75.218 ms
    tps = 2988.948548 (without initial connection time)

**7.** Как видим по тестам мы сильно упираемся в медленный HDD. Нас тормозит сброс кэша в диск и сброс wal.  
Следовательно для разгона нам лучше всего  
*а)* перенести на SSD  
*б)* wal файлы перенести на отдельный диск, т.к. случаная запись и последовательная очень не дружат.  
*в)* вынести темпы на отдельный диск или диск из памяти  
