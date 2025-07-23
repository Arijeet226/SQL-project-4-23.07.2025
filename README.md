# SQL-project-4-23.07.2025
# Background
In this SQL Portfolio Project | Flight Data Analysis in SQL, we will work on flight dataset with multiple tables and solve 8 very interesting SQL queries with complexity low, medium and high. We will learn many concepts like Case statement, Windows function, Joins, and many more. 
### The questions I wanted to answer through my SQL queries were:
## Q1 FIND THE BUSIEST AIRPORT BY THE NUMBER OF FLIGHTS TAKE OFF
```sql
SELECT a.name AS AIRPORT,
COUNT(*) AS TOTAL_FLIGHTS
FROM flights f
JOIN airports a
ON a.airportid=f.origin
GROUP BY A.name
ORDER BY TOTAL_FLIGHTS DESC
LIMIT 1;
```
## Q2 FIND THE TOTAL NUMBER OF TICKETS SOLD PER AIRLINES
```sql
SELECT al.name,Count(*) AS ticket_sold
FROM tickets t
JOIN flights f
ON t.FlightID=f.FlightID
JOIN airlines al
ON f.airlineid=al.airlineid
GROUP BY al.name;
```
## Q3 LIST ALL FLIGHTS OPERATED BY INDIGO WITH AIRPORT NAMES ORIGIN AND DESTINATION
```sql
SELECT f.flightid,a.name AS origin_airport,
a2.name AS destination_airport,f.departuretime,f.arrivaltime
FROM flights F
JOIN airlines al ON f.airlineid=al.airlineid
JOIN airports a ON f.origin=a.airportid
JOIN airports a2 ON f.destination=a2.airportid
WHERE al.name='IndiGo';
```
## Q4 FOR EACH AIRPORT SHOW THE TOP AIRLINE BY NUMBER OF FLIGHTS DEPARTING FROM THERE
```sql
SELECT 
    ap.name AS airport_name,
    al.name AS airline_name,
    fc.flight_count
FROM (
    SELECT 
        f.origin,
        f.airlineid,
        COUNT(*) AS flight_count,
        ROW_NUMBER() OVER (PARTITION BY f.origin ORDER BY COUNT(*) DESC) AS rn
    FROM flights f
    GROUP BY f.origin, f.airlineid
) fc
JOIN airports ap ON fc.origin = ap.airportid
JOIN airlines al ON fc.airlineid = al.airlineid
WHERE fc.rn = 1;
```
## Q5 FOR EACH FLIGHT SHOW TIME IN HOURS AND CATEGORY IT AS SHORT (<2H),MEDIUM(2-5 H),OR LONG (>5 H)
```sql
SELECT 
    flightid,
    departuretime,
    arrivaltime,
    EXTRACT(EPOCH FROM (arrivaltime - departuretime)) / 3600.0 AS durationhours,
    CASE
        WHEN EXTRACT(EPOCH FROM (arrivaltime - departuretime)) < 7200 THEN 'SHORT'
        WHEN EXTRACT(EPOCH FROM (arrivaltime - departuretime)) <= 18000 THEN 'MEDIUM'
        ELSE 'LONG'
    END AS flight_category
FROM flights;
```
## Q6 SHOW EACH PASSENGER'S FIRST AND LAST FLIGHTS DATES AND NUMBER OF FLIGHTS
```sql
SELECT 
    p.name,
    MIN(f.departuretime) AS first_flight,
    MAX(f.departuretime) AS last_flight,
    COUNT(*) AS total_flights
FROM tickets t
JOIN flights f ON t.flightid = f.flightid
JOIN passengers p ON t.passengerid = p.passengerid
GROUP BY t.passengerid, p.name
ORDER BY total_flights DESC;
```
## Q7 FIND FLIGHTS WITH THE HIGHEST PRICE TICKET SOLD FOR EACH ROUTE (origin->destination)
```sql
SELECT 
    a1.name AS origin,
    a2.name AS destination,
    rt.price,
    rt.ticketid
FROM (
    SELECT 
        f.flightid,
        f.origin,
        f.destination,
        t.ticketid,
        t.price,
        RANK() OVER (PARTITION BY f.origin, f.destination ORDER BY t.price) AS rnk
    FROM tickets t
    JOIN flights f ON t.flightid = f.flightid
) AS rt
JOIN airports a1 ON rt.origin = a1.airportid
JOIN airports a2 ON rt.destination = a2.airportid
WHERE rt.rnk = 1;
```
## Q8 FIND THE HIGHEST SPENDING PASSENGER IN EACH FREQUENT FLYER STATUS GROUP
```sql
SELECT 
    sub.name,
    sub.frequentflyerstatus,
    sub.total_spent
FROM (
    SELECT 
        p.passengerid,
        p.name,
        p.frequentflyerstatus,
        SUM(t.price) AS total_spent,
        RANK() OVER (PARTITION BY p.frequentflyerstatus ORDER BY SUM(t.price) DESC) AS rn
    FROM passengers p
    JOIN tickets t ON p.passengerid = t.passengerid
    GROUP BY p.passengerid, p.name, p.frequentflyerstatus
) AS sub
WHERE sub.rn = 1;
```
