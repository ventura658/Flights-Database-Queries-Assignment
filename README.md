# Flights Database Queries Assignment

## Introduction
This repository is for an assignment where you need to write and execute a series of SQL queries on the "flights" database. The database consists of several tables related to airports, airlines, routes, airplanes, passengers, and flights. The queries should be based on what you've learned in class, such as basic SELECT statements, JOINs, WHERE clauses, and GROUP BY for aggregation, while ensuring no duplicate records in the results.

The schema includes relationships between tables, and you'll need to write one SQL query for each specified question. Make sure to test the queries in your database environment.

## Database Schema
The "flights" database is composed of the following tables:

- **airports**  
  - `id` INT (primary key): The unique ID of the airport.  
  - `name` VARCHAR(45): The full name of the airport.  
  - `city` VARCHAR(45): The city where the airport is located.  
  - `country` VARCHAR(45): The country where the airport is located.  
  - `code` VARCHAR(45): The international code of the airport.

- **airlines**  
  - `id` INT (primary key): The unique ID of the airline.  
  - `name` VARCHAR(45): The full name of the airline.  
  - `alias` VARCHAR(45): The alternative name of the airline.  
  - `country` VARCHAR(45): The country of the airline's headquarters.  
  - `code` VARCHAR(45): The international code of the airline.  
  - `active` CHAR(1): Indicates if the airline is active ('Y' for active, 'N' for inactive).

- **routes**  
  - `id` INT (primary key): The unique ID of the route.  
  - `airlines_id` INT: The ID of the airline responsible for the route (foreign key to airlines.id).  
  - `source_id` INT: The ID of the departure airport (foreign key to airports.id).  
  - `destination_id` INT: The ID of the arrival airport (foreign key to airports.id).

- **airplanes**  
  - `id` INT (primary key): The unique ID of the airplane.  
  - `number` VARCHAR(45): The airplane's registration number.  
  - `manufacturer` VARCHAR(45): The manufacturing company of the airplane.  
  - `model` VARCHAR(45): The model of the airplane.

- **airlines_has_airplanes**  
  - `airlines_id` INT NOT NULL: The ID of the airline (foreign key to airlines.id, part of the primary key).  
  - `airplanes_id` INT NOT NULL: The ID of the airplane (foreign key to airplanes.id, part of the primary key).  
  - This is a junction table associating airlines with airplanes.

- **passengers**  
  - `id` INT (primary key): The unique ID of the passenger.  
  - `name` VARCHAR(45): The first name of the passenger.  
  - `surname` VARCHAR(45): The last name of the passenger.  
  - `year_of_birth` INT: The year of birth of the passenger.

- **flights**  
  - `id` INT (primary key): The unique ID of the flight.  
  - `routes_id` INT: The ID of the route for the flight (foreign key to routes.id).  
  - `date` DATE: The departure date of the flight.  
  - `airplanes_id` INT: The ID of the airplane for the flight (foreign key to airplanes.id).

- **flights_has_passengers**  
  - `flights_id` INT: The ID of the flight (foreign key to flights.id, part of the primary key).  
  - `passengers_id` INT: The ID of the passenger (foreign key to passengers.id, part of the primary key).  
  - This is a junction table associating flights with passengers.

For a visual representation of the er diagram, refer to the image below:  
![ER Diagram](https://github.com/ventura658/Flights-Database-Queries-Assignment/blob/15e5c6641bf3eb96dc7cd8942ba19282f969e2bf/Ex3-schnema.jpg)  

## Assignment Tasks
You need to write one SQL query for each of the following questions. Below are the provided queries for each question. Ensure the queries:
- Use only basic SQL features covered in class.
- Do not return duplicate records (as implemented in the queries).
- Test them in your database environment.
- 
[Flights Database](https://github.com/ventura658/Flights-Database-Queries-Assignment/blob/15e5c6641bf3eb96dc7cd8942ba19282f969e2bf/flights.sql)

1. **How many flights were carried out with 5 to 7 passengers from Athens?**
   
   SELECT COUNT(*) AS "#flights"
   FROM (
       SELECT fp.flights_id
       FROM airports a, routes r, flights f, flightshaspassengers fp
       WHERE (city = "Athens" OR city = "Αθήνα") 
       AND a.id = r.source_id
       AND r.id = f.routes_id
       AND f.id = fp.flights_id
       GROUP BY flights_id
       HAVING COUNT(DISTINCT passengers_id) BETWEEN 5 AND 7
   ) AS flights;

   2. **Find the manufacturer and model of the airplane that has made the most trips for Olympic Airways from Athens to London between 2011-02-01 and 2017-07-14.**
      
   SELECT ap.manufacturer, ap.model
   FROM routes r, airports a1, airports a2, flights f, airplanes ap, airlineshasairplanes aha, airlines al
   WHERE (r.source_id = a1.id AND a1.city = "Athens")
   AND (r.destination_id = a2.id AND a2.city = "London")        
   AND r.id = f.routes_id
   AND f.airplanes_id = ap.id
   AND ap.id = aha.airplanes_id
   AND aha.airlines_id = al.id 
   AND al.name = "Olympic Airways"
   AND f.date BETWEEN "2011-02-01" AND "2017-07-14"
   GROUP BY ap.manufacturer, ap.model
   ORDER BY COUNT(DISTINCT f.id) DESC
   LIMIT 1;

   3. **Find the names and surnames of passengers who made all their trips with British Airways.**

   SELECT DISTINCT p.name, p.surname 
   FROM passengers p, flightshaspassengers fhp, flights f, airplanes ap, airlineshasairplanes aha, airlines al
   WHERE p.id = fhp.passengers_id
   AND fhp.flights_id = f.id
   AND f.airplanes_id = ap.id
   AND ap.id = aha.airplanes_id
   AND aha.airlines_id = al.id 
   AND al.name = "British Airways";

   4. **For each airline that has more than 5 airplanes and operates more than 5 routes, find the name, code, number of airplanes, and number of routes.**

   SELECT al.name AS airline, aha.airlinesid, COUNT(DISTINCT aha.airplanesid) AS "#airplanes", COUNT(DISTINCT r.id) AS "#routes"
   FROM airlines al, routes r, airlineshasairplanes aha
   WHERE al.id = r.airlines_id
   AND aha.airlines_id = al.id 
   GROUP BY aha.airlines_id
   HAVING COUNT(DISTINCT r.id) > 5 AND COUNT(DISTINCT aha.airplanes_id) > 5;

   5. **Find the total visitors per airport where Aegean Airlines operates routes between 2011-01-01 and 2015-01-01.**
      
   SELECT a.name AS airportname, COUNT(DISTINCT fhp.passengersid) AS "#passengers"
   FROM flightshaspassengers fhp, flights f, routes r, airlines al, airports a
   WHERE fhp.flights_id = f.id
   AND f.routes_id = r.id
   AND r.airlines_id = al.id
   AND (r.sourceid = a.id OR r.destinationid = a.id)
   AND al.name = "Aegean Airlines"
   AND f.date BETWEEN "2011-01-01" AND "2015-01-01"
   GROUP BY a.name;

   6. **Find how many passengers traveled from Athens to any destination more than 5 times between 2010-01-01 and 2015-01-01.**
      
   SELECT COUNT(*) AS "#passengers"
   FROM(
       SELECT fhp.passengersid, COUNT(DISTINCT fhp.flightsid) AS "#passengers", a.city
       FROM flightshaspassengers fhp, flights f, routes r, airports a
       WHERE fhp.flights_id = f.id
       AND f.routes_id = r.id
       AND r.source_id = a.id
       AND city = "Athens"
       GROUP BY fhp.passengers_id
       HAVING COUNT(DISTINCT fhp.flights_id) > 5
       ORDER BY 2 DESC
   ) AS sub;

   7. **Find the airport with the most routes with different active airlines.**
      
   SELECT ap.name AS airport_name, COUNT(DISTINCT r.id) AS "#routes", 
          COUNT(DISTINCT r.airlinesid) AS "#activeairlines"
   FROM airports ap, routes r, airlines al
   WHERE (r.sourceid = ap.id OR r.destinationid = ap.id)
   AND al.id = r.airlines_id
   AND al.active = "Y"
   GROUP BY ap.name
   ORDER BY 2 DESC, 3 DESC
   LIMIT 1;

 8. **Find the airline with the most travelers under 25 years old.**  

   SELECT al.name AS airlines, COUNT(DISTINCT fhp.passengersid) AS "U25passengers"
   FROM airlines al, flightshaspassengers fhp, passengers p, flights f, routes r
   WHERE p.id = fhp.passengers_id 
   AND fhp.flights_id = f.id
   AND f.routes_id = r.id
   AND r.airlines_id = al.id
   AND (2024 - yearofbirth) < 25
   GROUP BY al.name
   ORDER BY 2 DESC
   LIMIT 1;

  9. **Find how many flights were executed by each airline with aircraft from the manufacturer Boeing.**

   SELECT * FROM (
       SELECT al.name, COUNT(DISTINCT f.id) AS flight_count
       FROM airlines al, flights f, routes r, airlineshasairplanes aha, airplanes a
       WHERE al.id = r.airlines_id
       AND r.id = f.routes_id
       AND aha.airlines_id = al.id
       AND aha.airplanes_id = a.id
       AND a.manufacturer = 'Boeing'
       GROUP BY al.name

       UNION ALL

       SELECT al.name, SUM(ar.manufacturer = 'Boeing') AS boeingflightcount
       FROM airlines al
       LEFT JOIN routes r ON r.airlines_id = al.id
       LEFT JOIN flights f ON f.routes_id = r.id
       LEFT JOIN airlineshasairplanes aha ON al.id = aha.airlines_id
       LEFT JOIN airplanes ar ON aha.airplanes_id = ar.id
       WHERE al.id NOT IN (
           SELECT DISTINCT al.id
           FROM airlines al
           JOIN routes r ON r.airlines_id = al.id
           JOIN airlineshasairplanes aha ON al.id = aha.airlines_id
           JOIN airplanes ar ON aha.airplanes_id = ar.id
           WHERE ar.manufacturer = 'Boeing'
       )
       GROUP BY al.name
   ) AS sub
   ORDER BY sub.name ASC;

   10. **Find the 5 airlines that passengers prefer the least for flights to and from London.**

    SELECT al.name, COUNT(DISTINCT fhp.passengers_id) AS "#passengers", 
           COUNT(DISTINCT f.id) AS "#flights"
    FROM airlines al, flightshaspassengers fhp, routes r, airports ap, flights f
    WHERE al.id = r.airlines_id
    AND (r.sourceid = ap.id OR r.destinationid = ap.id)
    AND fhp.flights_id = f.id
    AND f.routes_id = r.id
    AND ap.city = "London"
    GROUP BY al.name
    HAVING COUNT(DISTINCT f.id) > 0
    ORDER BY 2 ASC, 3 DESC
    LIMIT 5;
