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
);```


