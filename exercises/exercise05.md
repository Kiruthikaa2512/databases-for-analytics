# Exercise 05: SQLDA Database - Dates, Data Quality, Arrays, and JSON

- Name:Kiruthikaa
- Course: Database for Analytics
- Module: 5
- Database Used:  `sqlda` (Sample Datasets)
- Tools Used: PostgreSQL (pgAdmin or psql)

---

## Instructions

- Use the **sqlda** database from the "Loading the Sample Datasets" instructions.
- For each SQL task:
  - Include your SQL in a fenced code block
  - Execute it and include a **screenshot** showing the query and results
- Store screenshots in the `screenshots/` folder and embed them below each answer.
- For explanation questions:
  - Write your answer in complete sentences
  - Include a screenshot if requested

---

## Question 1

Using the `sqlda` database, write the SQL needed to show a **list of years** that emails were sent.

Your results should list years like this (order matters):

```
year
2011
2013
2014
2015
2016
2017
2018
2019
```

### SQL

```sql
SELECT DISTINCT EXTRACT(YEAR FROM sent_date) AS year
FROM emails
WHERE sent_date IS NOT NULL
ORDER BY year;
```

### Screenshot

![Q1 Screenshot](screenshots/q1_email_years.png)

---

## Question 2

Using the `sqlda` database, write the SQL needed to show the **number of messages sent by year**, ordered by year (as shown in the prompt).

Output should resemble:

```
count   year
...
```

### SQL

```sql
SELECT 
    COUNT(*) AS count,
    EXTRACT(YEAR FROM sent_date) AS year
FROM emails
WHERE sent_date IS NOT NULL
GROUP BY year
ORDER BY year;
```

### Screenshot

![Q2 Screenshot](screenshots/q2_message_count_by_year.png)

---

## Question 3

Using the `sqlda` database, write the SQL needed to show:
- the **sent date**
- the **opened date**
- the **interval** between the two

Only include emails that contain **both** a sent date and an opened date.

### SQL

```sql
SELECT 
    sent_date,
    opened_date,
    opened_date - sent_date AS interval
FROM emails
WHERE sent_date IS NOT NULL
  AND opened_date IS NOT NULL;
```

### Screenshot

![Q3 Screenshot](screenshots/q3_sent_opened_interval.png)

---

## Question 4

Using the `sqlda` database, write the SQL needed to show emails that contain an **opened date BEFORE the sent date**.

### SQL

```sql
SELECT 
    sent_date,
    opened_date
FROM emails
WHERE opened_date < sent_date;
```

### Screenshot

![Q4 Screenshot](screenshots/q4_opened_before_sent.png)

---

## Question 5

Using the `sqlda` database: there are **over 100 emails** that contain an opened date **BEFORE** the sent date.

After looking at the data, **why is this the case?**

### Answer

When I looked through the records where the opened date shows up earlier than the sent date, a pattern stood out right away. All of the sent_date values use the exact same time of day - 15:00:00 which doesn’t look like a real event timestamp. It feels more like a default or may be a placeholder time that was assigned during data entry or import. The opened_date values, on the other hand, have natural‑looking times that vary throughout the day. Because the sent timestamps aren’t accurate, comparing them to the opened timestamps creates situations where it looks like the email was opened before it was sent. So the issue isn’t with the logic, it’s simply a data quality problem caused by inconsistent timestamp recording.


### Screenshot (if requested by instructor)

![Q5 Screenshot](screenshots/q5_explain_date_issue.png)

---

## Question 6

Using the `sqlda` database, explain in your own words what the following code does:

```sql
CREATE TEMP TABLE customer_points AS (
    SELECT
        customer_id,
        point(longitude, latitude) AS lng_lat_point
    FROM customers
    WHERE longitude IS NOT NULL
    AND latitude IS NOT NULL
);

CREATE TEMP TABLE dealership_points AS (
    SELECT
        dealership_id,
        point(longitude, latitude) AS lng_lat_point
    FROM dealerships
);

CREATE TEMP TABLE customer_dealership_distance AS (
    SELECT
       customer_id,
       dealership_id,
       c.lng_lat_point <@> d.lng_lat_point AS distance
    FROM customer_points c
    CROSS JOIN dealership_points d
);
```

### Answer

This code creates three temporary tables, each with a specific purpose.
The first temp table, customer_points, stores two columns: the customer_id and a combined latitude and longitude value for customers who actually have coordinates.
The second temp table, dealership_points, does the same thing for dealerships, with dealership_id and their coordinate value.

Once those two tables are set up, the last temp table, customer_dealership_distance, takes every customer from the first table and matches them with every dealership from the second table. For each pair, it calculates the distance between their coordinates using the <@> operator from the earthdistance extension. The result is a list of all customer - dealership combinations along with how far apart they are.


---

## Question 7

Using the `sqlda` database, write SQL to display an **array of salespeople for each dealership**, sorted by dealership.

For example - dealership 1 is below:

```text
"{""Fidell,Granville"",""Onele,Jereme"",""Sheriff,Lelia"",""McSpirron,Massimiliano"",""Rennick,Nadia"",""Mace,Eveleen"",""Oxteby,Dukie"",""Spong,Marcos"",""Wogden,Quent"",""Duny,Sandye"",""Loraine,Englebert"",""Meere,Ira"",""Gibbens,Cristine"",""Prine,Lyda"",""McCoughan,Sheff"",""Schule,Giselbert"",""McAndie,Eleen"",""Dosedale,Dorie"",""Nafziger,Shay""}"
```

### SQL

```sql
SELECT 
    dealership_id,
    ARRAY_AGG(
        last_name || ',' || first_name
        ORDER BY last_name, first_name
    ) AS salespeople
FROM salespeople
GROUP BY dealership_id
ORDER BY dealership_id;
```

### Screenshot

![Q7 Screenshot](screenshots/q7_salespeople_array_by_dealership.png)

---

## Question 8

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state** of the dealership
- the **number of salespeople** for the dealership

Sort by **state**.

Reference image:

![05-ExerciseArray](./instructions/05-ExerciseArray.jpg)

### SQL

```sql
SELECT 
    d.state,
    sp.dealership_id,
    ARRAY_AGG(
        sp.last_name || ',' || sp.first_name
        ORDER BY sp.last_name, sp.first_name
    ) AS salespeople,
    COUNT(*) AS number_of_salespeople
FROM salespeople sp
JOIN dealerships d 
    ON sp.dealership_id = d.dealership_id
GROUP BY d.state, sp.dealership_id
ORDER BY d.state;
```

### Screenshot

![Q8 Screenshot](screenshots/q8_salespeople_array_state_count.png)

---

## Question 9

Using the `sqlda` database, write the SQL needed to convert the **customers** table to **JSON**.

### SQL

```sql
SELECT json_agg(c)
FROM customers c;
```

### Screenshot

![Q9 Screenshot](screenshots/q9_customers_to_json.png)

---

## Question 10

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state**
- the **number of salespeople**
- sorted by **state**

Then **convert this result to JSON**.

Reference image:

![05-ExerciseArray-1](./instructions/05-ExerciseArray-1.jpg)

### SQL

```sql
SELECT json_agg(result)
FROM (
    SELECT 
        d.state,
        sp.dealership_id,
        ARRAY_AGG(
            sp.last_name || ',' || sp.first_name
            ORDER BY sp.last_name, sp.first_name
        ) AS salespeople,
        COUNT(*) AS number_of_salespeople
    FROM salespeople sp
    JOIN dealerships d 
        ON sp.dealership_id = d.dealership_id
    GROUP BY d.state, sp.dealership_id
    ORDER BY d.state
) AS result;
```

### Screenshot

![Q10 Screenshot](screenshots/q10_salespeople_array_to_json.png)
