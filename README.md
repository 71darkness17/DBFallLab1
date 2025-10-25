# Lab 1
    by Zavorokhin Vladimir

## Task 1
```sql
SELECT *
FROM flights
WHERE arrival_airport = 'SVO'
  AND actual_arrival::date = '2017-07-22'
  AND actual_arrival::time BETWEEN '16:00' AND '19:00';
```
Using `SELECT` to select what we need 
Using `WHERE` to filter data and `AND` and `BETWEEN` to set multiple filtering
## Task 2
```sql
UPDATE flights
SET scheduled_departure = scheduled_departure + INTERVAL '30 minutes',
    scheduled_arrival   = scheduled_arrival + INTERVAL '30 minutes'
WHERE scheduled_departure BETWEEN '2017-09-11 23:00:00+00' AND '2017-09-12 06:00:00+00';
```
Using `UPDATE` to change values in table, `INTERAVL` to convert into timestamp and move not only departure time, cause we have `flight_check` constraint
## Task 3
```sql
SELECT
    departure_airport,
    COUNT(*) AS total_flights,
    AVG(scheduled_arrival - scheduled_departure) AS avg_duration
FROM bookings.flights
GROUP BY departure_airport
ORDER BY total_flights DESC;
```
Using `COUNT(*)` and `AVG()` to find targets
and `GROUP BY` and `ORDER_BY` to make right order in list
# Lab 2

We need to find tickets with 2 flights and select arrival of first flight and departure of the second one. Define `transit_arrival` as an arrival of the first flight and `transit_departure` as a departure of second flight. Use `ROW_NUMBER()` to numerate flights, ordered by departure and then choose `transit_arrival` and `transit_departure` by using `MAX` after filtering by `CASE/WHEN/THEN/END`. After that count transit passagers by using `COUNT(transit_arrivals)` and calculate average transit time by using `AVG()`. Group by airport and order from the most used for transit to the least.

```sql
SELECT
    transit_airport,
    COUNT(transit_arrival) AS transit_count,
    AVG(transit_departure - transit_arrival) AS avg_transit_time
FROM (
    SELECT
    ticket_no,
    MAX(CASE WHEN rn = 1 THEN scheduled_arrival END) AS transit_arrival,
    MAX(CASE WHEN rn = 2 THEN scheduled_departure END) AS transit_departure,
    MAX(CASE WHEN rn = 1 THEN arrival_airport END) AS transit_airport
    FROM (
        SELECT
            tf.ticket_no,
            tf.flight_id,
            f.scheduled_departure,
            f.scheduled_arrival,
            f.arrival_airport,
            ROW_NUMBER() OVER (PARTITION BY tf.ticket_no ORDER BY f.scheduled_departure) AS rn
        FROM ticket_flights tf
        JOIN flights f ON f.flight_id = tf.flight_id
    ) t
    GROUP BY ticket_no
    HAVING COUNT(*) = 2 
) h
GROUP BY transit_airport
ORDER BY transit_count DESC;
```

__Data Sample__

 transit_airport | transit_count |    avg_transit_time
-----------------|---------------|-------------------------
 DME             |         28559 | 7 days 28:01:24.078575
 SVO             |         24414 | 7 days 26:29:13.538953
 OVB             |         12146 | 7 days 32:38:14.829573
 AER             |         10649 | 6 days 37:35:03.944032
 VKO             |          7148 | 7 days 24:31:39.468383
 SVX             |          5889 | 7 days 34:36:59.612837
 LED             |          4998 | 7 days 12:26:35.738296
 KGD             |          4755 | 7 days 26:29:06.750788
 KRR             |          4643 | 7 days 33:47:57.10532
 KHV             |          3805 | 6 days 22:56:12.930354
 KUF             |          3425 | 7 days 29:17:26.10219
 ROV             |          3110 | 7 days 30:51:28.553055
 KJA             |          3105 | 7 days 30:20:38.840579
 BZK             |          3068 | 7 days 20:38:09.113429
 UFA             |          2993 | 7 days 42:38:38.409622
 MCX             |          2662 | 6 days 09:51:23.50864

