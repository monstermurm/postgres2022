**Все примеры построены на тестовой базе полетов для PostgreSQL.**  

**1.** Создаем индекс и запрос использующий созданный индекс.  
создание запроса:  
*create index airports_airport_code_ix on bookings.airports (airport_code);*  

запрос  
     select  
        *  
    from  
        bookings.airports  
    where  
        airport_code = 'AER'  
        
план выполнения:  
      Index Scan using airports_airport_code_ix on airports as airports (rows=1 loops=1)
     Index Cond: (airport_code = 'AER'::bpchar)
     
Индекс может использоваться для быстрого поиска по коду и для связи по fk с другими таблицами.
     
**2.** Создаем индекс полнотекстового поиска.  
*create index passenger_name_ix on bookings.tickets using gin (to_tsvector('english', passenger_name));*

Индекс может использоваться для неточного поиска пассажира по Фамилии, Имени, части их.

**3.** Создаем индекс на часть таблицы.  
*create index status_ix on bookings.flights (status) where status = 'Scheduled';*

Индекс может использоваться для быстрого отслеживания рейсов в пути, например онлайн табло.

**4.** Создаем составной индекс.  
*create index flight_no_departure_airport_ix on bookings.flights (flight_no, departure_airport);*

Индекс может использоваться для получения данных по вылетам рейса из конкретного аэропорта.

Не очень понимаю что нужно написать о том, что и как делал. Создание индекса определяется совокупностью таких параметров как частота запроса с нужным отбором и селективностью столбца по которому строится отбор. Если эта совокупность позволяет существенно поднять производительность - смысл в индексе имеет место быть. Проблем тут никаких не возникло.

**5.** Пример прямого запроса  
select
    f.flight_no,
    f.scheduled_departure,
    a.airport_name
from
    bookings.flights as f
    inner join bookings.airports as a
        on f.departure_airport = a.airport_code;
        
выведем информацию и запланированных вылетах с названием аэропортов.

**6.** Пример левостороннего запроса  
select
    a.airport_code,
    a.airport_name  
from
    bookings.airports as a
    left outer join bookings.flights as f
        on a.airport_code = f.departure_airport
        and f.scheduled_departure > '20170101'
where
    f.flight_id is null;
    
Выведем информацию из каких аэропортов нет рейсов после 01.01.2017.
    
**7.** Пример кросс запроса  
select
    aircraft_code,
    airport_code
from
    bookings.aircrafts,
    bookings.airports;
    
Выведем все комбинации номер борта + код аэропорта.

**8.** Пример полного запроса  
select
    coalesce(f.d, t.d) as d,
    SUM(CASE WHEN f.flight_id is null THEN 0 ELSE 1 END) as s
from
    (select 
        flight_id,
        date_trunc('DAY', scheduled_departure) as d
    from
        bookings.flights
    where
        scheduled_departure between '20161101' and '20161130'
        and date_trunc('DAY', scheduled_departure) not in ('20161104', '20161122')
    ) as f
    full join
    (select
        date '20161101' + t2.i * 10 + t1.i as d
    from
        (select 0 as i union all select 1 union all select 2 union all select 3 union all select 4 union all select 5
        union all select 6 union all select 7 union all select 8 union all select 9) as t1,
        (select 0 as i union all select 1 union all select 2 union all select 3 union all select 4 union all select 5
        union all select 6 union all select 7 union all select 8 union all select 9) as t2
    where
        date '20161101' + t2.i * 10 + t1.i between '20161101' and '20161130'
        and date '20161101' + t2.i * 10 + t1.i not in ('20161105', '20161118')
     ) as t
        on f.d = t.d
group by
    coalesce(f.d, t.d);

Подсчет количества рейсов по датам с заданной матрицей дат учета.
