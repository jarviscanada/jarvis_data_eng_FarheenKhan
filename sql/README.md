# Introduction
This project aims to develop a SQL relational database for a newly established city club. The database comprises three key tables, housing data related to club members, facilities, and bookings. This database can be used to conduct data analysis through SQL queries to gain insights into customer dynamics and the demand for various facilities within the club.<br>
The codes below illustrate how three tables are created in DDL (Data Definition Language), each with its attributes, data types and constraints, as well as specified primary keys and foreign keys to define their relationships. The following queries are examples of how the club can utilize data to answer business questions.
# SQL Queries
```
Table Setup (DDL)

CREATE TABLE cd.members (
    memid INT NOT NULL,
    surname VARCHAR(200) NOT NULL,
    firstname VARCHAR(200) NOT NULL,
    address VARCHAR(300) NOT NULL,
    zipcode INT NOT NULL,
    telephone VARCHAR(20) NOT NULL,
    recommendedby INT,
    joindate TIMESTAMP NOT NULL,
    CONSTRAINT members_pk PRIMARY KEY (memid),
    CONSTRAINT members_recommendedby_fk FOREIGN KEY (recommendedby) REFERENCES cd.members(memid) ON DELETE SET NULL 
);

CREATE TABLE cd.facilities (
    facid INT NOT NULL,
    name VARCHAR(100) NOT NULL,
    membercost NUMERIC NOT NULL,
    initialoutlay NUMERIC NOT NULL,
    monthlymaintenance NUMERIC NOT NULL,
    CONSTRAINT facs_pk PRIMARY KEY (facid)
);

CREATE TABLE cd.bookings (
    bookid INT NOT NULL,
    facid INT NOT NULL,
    memid INT NOT NULL,
    starttime TIMESTAMP NOT NULL,
    slots INT NOT NULL,
    CONSTRAINT bookings_pk PRIMARY KEY (bookid),
    CONSTRAINT bookings_memid_fk FOREIGN KEY (memid) REFERENCES cd.members(memid),
    CONSTRAINT facs_facid_fk FOREIGN KEY (facid) REFERENCES cd.facilities(facid)
);
```

Q1 :
Add a new spa into the facilities table.
```
INSERT INTO cd.facilities
    (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
VALUES (9, 'Spa', 20, 30, 100000, 800);
```
Q2:Add a new spa into the facilities table and automatically generate the value for the next facid, instead of specifying it as a constant.
```
INSERT INTO cd.facilities
    (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
SELECT (SELECT MAX(facid) FROM cd.facilities) + 1, 'Spa', 20, 30, 100000, 800;
```
Q3:Update the initial outlay of the second tennis court, which was 10000 rather than 8000.
```
UPDATE cd.facilities
SET initialoutlay = 10000
WHERE name = 'Tennis Court 2';
```
Q4:Update the price of the second tennis court so that it costs 10% more than the first one.
```
UPDATE cd.facilities AS f
SET
    membercost = (SELECT membercost * 1.1 FROM cd.facilities WHERE facid = 0),
    guestcost = (SELECT guestcost * 1.1 FROM cd.facilities WHERE facid = 0)
WHERE f.facid = 1; 
```
Q5:Delete all bookings from the cd.bookings table.
```
DELETE FROM cd.bookings;
```
Q6:Remove member 37, who has never made a booking, from the cd.members table.
```
DELETE FROM cd.members
WHERE memid = 37;
```
Q7:Return facid, facility name, member cost, and monthly maintenance of the facilities that charge a fee to members, and that fee is less than 1/50th of the monthly maintenance cost.
```
SELECT facid, name, membercost, monthlymaintenance
FROM cd.facilities
WHERE membercost > 0 AND membercost < monthlymaintenance/50; 
```
Q8:Return a list of all facilities with the word 'Tennis' in their name.
```
SELECT *
FROM cd.facilities
WHERE name LIKE '%Tennis%'; 
```
Q9:Retrieve the details of facilities with ID 1 and 5 without using the OR operator.
```
SELECT *
FROM cd.facilities
WHERE facid IN (1,5);
```
Q10:Return the memid, surname, firstname, and joindate of the members who joined after the start of September 2012.
```
SELECT memid, surname, firstname, joindate
FROM cd.members
WHERE joindate > '2012-08-31'; 
```
Q11:Create a combined list of all surnames and all facility names.
```
SELECT surname FROM cd.members
        UNION
        SELECT name FROM cd.facilities; 
```
Q12:Return a list of the start times for bookings by members named 'David Farrell'.
```
SELECT starttime
FROM cd.members AS m
LEFT JOIN cd.bookings AS b
ON m.memid = b.memid
WHERE firstname = 'David' AND surname = 'Farrell'; 
```
Q13:Return a list of start time and facility name pairings for the date '2012-09-21', ordered by the time.
```
SELECT b.starttime AS start, f.name AS name
FROM cd.bookings AS b
LEFT JOIN cd.facilities AS f
ON b.facid = f.facid
WHERE f.name IN ('Tennis Court 1', 'Tennis Court 2') AND
    b.starttime >= '2012-09-21' AND
    b.starttime < '2012-09-22'
ORDER BY b.starttime;
```
Q14:Return a list of all members, including the individual who recommended them (if any).
```
SELECT m1.firstname AS memfname,
       m1.surname AS memsname,
       m2.firstname AS recfname,
       m2.surname AS recsname
FROM cd.members AS m1
LEFT JOIN cd.members AS m2
ON m2.memid = m1.recommendedby
ORDER BY m1.surname, m1.firstname
```
Q15:Return a list of all members who have recommended another member with no duplicates in the list. Order the result by surname and firstname.
```
SELECT m2.firstname AS firstname, m2.surname AS surname
FROM (
    SELECT DISTINCT(recommendedby)
    FROM cd.members AS m1
    WHERE m1.recommendedby IS NOT NULL
    ) AS m1
LEFT JOIN cd.members AS m2
ON m1.recommendedby = m2.memid
ORDER BY surname, firstname;
```
Q16:Return a list of all members, including the individual who recommended them (if any), without using any joins and no duplicates.
```S
ELECT DISTINCT(mem.firstname || ' ' || mem.surname) AS member,
    (SELECT ref.firstname || ' ' || ref.surname AS recommender
    FROM cd.members AS ref
    WHERE mem.recommendedby = ref.memid)
FROM cd.members AS mem
ORDER BY member; 
```
Q17:Return a count of the number of recommendations each member has made, ordered by member ID.
```
SELECT recommendedby, COUNT(recommendedby) AS count
FROM cd.members
WHERE recommendedby IS NOT NULL
GROUP BY recommendedby
ORDER BY recommendedby;
```
Q18:Return a list of facility id with corresponding slots booked, ordered by facility id.
```
SELECT facid, SUM(slots) AS "Total Slots"
FROM cd.bookings
GROUP BY facid
ORDER BY facid;
```
Q19:Return a list of facility id with corresponding slots booked in September 2012, sorted by the number of slots.
```
SELECT facid, SUM(slots) AS "Total Slots"
FROM cd.bookings
WHERE starttime >= '2012-09-01' AND starttime < '2012-10-01'
GROUP BY facid
ORDER BY SUM(slots);
```
Q20:Return a list of facility id with corresponding slots booked per month in 2012, sorted by the id and month
```
SELECT facid, EXTRACT(MONTH FROM starttime) AS month, SUM(slots) AS "Total Slots"
FROM cd.bookings
WHERE EXTRACT(YEAR FROM starttime) = 2012
GROUP BY facid, month
ORDER BY facid, month
```
Q21:Print the total number of members (including guests) who have made at least one booking.
```
SELECT COUNT(DISTINCT memid)
FROM cd.bookings;
```
Q22:Produce a list of each member name, id, and their first booking after September 1st 2012, ordered by member ID.
```
SELECT DISTINCT m.surname, m.firstname, m.memid, MIN(b.starttime)
FROM cd.members AS m
LEFT JOIN cd.bookings AS b
ON m.memid = b.memid
WHERE b.starttime >= '2012-09-01'
GROUP BY m.memid
ORDER BY m.memid;
```
Q23:Return a list of all member names, with each row containing the total member count, ordered by join date..
```
SELECT COUNT(*) OVER(), firstname, surname
FROM cd.members
ORDER BY joindate;
```
Q24:Create an increasing numbered list of members (including guests), ordered by join date.
```
SELECT ROW_NUMBER() OVER (ORDER BY joindate),
    firstname, surname
FROM cd.members;
```
Q25:Display the facility id that has the highest number of slots booked.
```
SELECT facid, SUM(slots) AS total
FROM cd.bookings
GROUP BY facid
ORDER BY total DESC
LIMIT 1
```
Q26:Display the names of all members, formatted as 'Surname, Firstname'.
```
SELECT surname || ', ' || firstname
FROM cd.members;
```
Q27:Return all the member ID and telephone number that contain parentheses, sorted by member ID.
```
SELECT memid, telephone
FROM cd.members
WHERE telephone ~ '[()]';
```
Q28:Count the number of members whose surname starts with each letter of the alphabet, sorted by the letter.
```
SELECT SUBSTRING(surname,1,1) AS letter, COUNT(*)
FROM cd.members
GROUP BY letter
ORDER BY letter;
```









