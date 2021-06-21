# Аналитика в авиакомпании

## Данные
Входные данные — база данных об авиаперевозках:
- информация об аэропортах:
    - трёхбуквенный код аэропорта;
    - название аэропорта;
    - город;
    - часовой пояс;
- информация о самолётах:
    - код модели самолёта;
    - модель самолёта;
    - дальность полётов;
- информация о билетах:
    - уникальный номер билета;
    - уникальный идентификатор пассажира;
    - имя и фамилия пассажира;
- информация о рейсах:
    - уникальный идентификатор рейса;
    - аэропорт вылета;
    - дата и время вылета;
    - аэропорт прилёта;
    - дата и время прилёта;
    - уникальный идентификатор самолёта;
- стыковая таблица «рейсы-билеты»:
    - номер билета;
    - уникальный идентификатор рейса;
- информация о фестивалях:
    - уникальный номер фестиваля;
    - дата проведения фестиваля;
    - город проведения фестиваля;
    - название фестиваля.

## Задачи
- **Образовательная** - отработать навыки парсинга сайтов *(код под текстом)*, выгрузки баз данных посредством SQL *(код под текстом)*, предварительного анализа и визуализации данных.
- **Практическая** - проанализировать спрос пассажиров на рейсы в города, где проходят крупнейшие фестивали.

## Используемые библиотеки
- `pandas`
- `matplotlib`
- `requests`
- `bs4`


```python
import pandas as pd
import requests
from bs4 import BeautifulSoup

URL='~~festival_news/index.html'
req = requests.get(URL)
soup = BeautifulSoup(req.text, 'lxml')
table = soup.find('table')

heading_table = [] 
for row in table.find_all('th'):
    heading_table.append(row.text)

content = []
for row in table.find_all('tr'):
    if not row.find_all('th'):
        content.append([element.text for element in row.find_all('td')]) 
        
festivals = pd.DataFrame(content, columns=heading_table)

print(festivals)
```

```SQL
SELECT
    departure_airport AS departure_airport,
    COUNT(flight_id) AS cnt_flights
FROM
    flights
GROUP BY
    departure_airport
ORDER BY
    cnt_flights DESC;
```

```SQL
SELECT
    aircrafts.model AS model,
    COUNT(flights.flight_id) AS flights_amount    
FROM
    flights
INNER JOIN
    aircrafts ON aircrafts.aircraft_code = flights.aircraft_code
WHERE
    EXTRACT(MONTH FROM departure_time) = 9
GROUP BY
    aircrafts.model;
```

```SQL
SELECT
    SUBQ.type_aircraft AS type_aircraft,
    COUNT(flights.flight_id) AS flights_amount    
FROM
    flights
INNER JOIN
    (SELECT
        aircraft_code,
        CASE WHEN model LIKE '%Boeing%' THEN 'Boeing'
        WHEN model LIKE '%Airbus%' THEN 'Airbus'
        ELSE 'other' END AS type_aircraft
    FROM
        aircrafts) AS SUBQ ON SUBQ.aircraft_code = flights.aircraft_code
WHERE
    EXTRACT(MONTH FROM flights.departure_time) = 9
GROUP BY
    SUBQ.type_aircraft;
```

```SQL
SELECT
    SUBQ.city AS city,
    AVG(SUBQ.cnt) AS average_flights
FROM
    (SELECT
        airports.city AS city,
        DATE_TRUNC('day', flights.arrival_time) AS trunc_date,
        COUNT(flights.flight_id) AS cnt
    FROM
        flights
    INNER JOIN airports ON airports.airport_code = flights.arrival_airport
    WHERE
        EXTRACT(MONTH FROM flights.arrival_time) = 8
    GROUP BY
        airports.city,
        DATE_TRUNC('day', flights.arrival_time)) AS SUBQ
GROUP BY
        SUBQ.city;
```

```SQL
SELECT
    festival_name AS festival_name,
    EXTRACT(WEEK FROM festival_date) AS festival_week
FROM
    festivals
WHERE
    festival_city = 'Москва' AND DATE_TRUNC ('day', festival_date) BETWEEN '2018-06-23' AND '2018-09-30';
```

```SQL
SELECT 
    SUBQ.week_number,
    SUBQ.ticket_amount,
    SUBQ.festival_week,
    SUBQ.festival_name
FROM 
    (
    (SELECT
        EXTRACT (week FROM flights.departure_time) AS week_number,
        COUNT(ticket_flights.ticket_no) AS ticket_amount
    FROM 
        airports
    INNER JOIN 
        flights 
    ON 
        airports.airport_code = flights.arrival_airport
    INNER JOIN 
        ticket_flights 
    ON 
        flights.flight_id = ticket_flights.flight_id
    WHERE 
        airports.city = 'Москва' AND CAST(flights.departure_time AS date) BETWEEN '2018-07-23' AND '2018-09-30'
    GROUP BY
        week_number
    ) AS subq1
    LEFT JOIN 
        (SELECT         
            festival_name,  
            EXTRACT (week FROM festivals.festival_date) AS festival_week
        FROM 
            festivals
        WHERE
            festival_city = 'Москва' AND CAST(festivals.festival_date AS date) BETWEEN '2018-07-23' AND '2018-09-30'
        ) subq2 ON subq1.week_number = subq2.festival_week
) AS SUBQ;
```