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

**8.** 
