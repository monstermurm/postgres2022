**1.** Возьмем самую большую таблицу из базы ticket_flights > 500 MB  
*CREATE TABLE IF NOT EXISTS bookings.ticket_flights_part  
(  
    ticket_no character(13) COLLATE pg_catalog."default" NOT NULL,  
    flight_id integer NOT NULL,  
    fare_conditions character varying(10) COLLATE pg_catalog."default" NOT NULL,  
    amount numeric(10,2) NOT NULL  
) PARTITION by RANGE (flight_id)*  
Создавать индексы и констраны не будем, это увеличит время заполнения, их можно создать потом

**2.** Определим границы для секций  
*select MIN(flight_id), MAX(flight_id) from bookings.ticket_flights*  
    "min"	"max"
    2	214867

**3.** Исходя из текущего диапазона, разобьем таблицу по 50 тыс строк на 4 полные секции и одну по умолчанию  
*CREATE TABLE bookings.ticket_flights_part_0_50 PARTITION OF bookings.ticket_flights_part  
    FOR VALUES FROM (0) TO (50000);  
CREATE TABLE bookings.ticket_flights_part_50_100 PARTITION OF bookings.ticket_flights_part  
    FOR VALUES FROM (50000) TO (100000);  
CREATE TABLE bookings.ticket_flights_part_100_150 PARTITION OF bookings.ticket_flights_part  
    FOR VALUES FROM (100000) TO (150000);  
CREATE TABLE bookings.ticket_flights_part_150_200 PARTITION OF bookings.ticket_flights_part  
    FOR VALUES FROM (150000) TO (200000);  
CREATE TABLE bookings.ticket_flights_part_DEFAULT PARTITION OF bookings.ticket_flights_part  
    DEFAULT;*  

**4.** Заполняем таблицу.  
*insert into bookings.ticket_flights_part select * from bookings.ticket_flights;*  

    INSERT 0 8391852
    Запрос завершён успешно, время выполнения: 16 secs 335 msec.
