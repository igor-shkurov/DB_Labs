# Databases Lab 1
Students: Filip Nikolic, Igor Shkurov

## Assignment 1

> 1. Write SQL-statements that create the corresponding tables. Come up with reasonable constraints and datatypes for the fields of the tables.

<div style="text-align: left">We started with creation of the Program table, since it does not depend on the other tables. All primary key fields were chosen to be serial, which allows automatic insertion and incrementation of those values:</div>

```
CREATE TABLE _PROGRAM (
    programID SERIAL PRIMARY KEY,
    _name VARCHAR(100) NOT NULL UNIQUE,
    requiredCPs INT NOT NULL CHECK (requiredCPs > 0)
);
```

<div style="text-align: left">The Student table:</div>

```
CREATE TABLE STUDENT (
    studentID SERIAL PRIMARY KEY,
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    dob DATE NOT NULL,
    programID INT,
    FOREIGN KEY (programID) REFERENCES _PROGRAM(programID)
);
```
<div style="text-align: left">The Course table:</div>

```
CREATE TABLE COURSE (
    courseID SERIAL PRIMARY KEY,
    _name VARCHAR(5) NOT NULL,
    description TEXT,
    creditPoints INT NOT NULL CHECK (creditPoints > 0),
    programID INT,
    FOREIGN KEY (programID) REFERENCES _PROGRAM(programID)
);
```
<div style="text-align: left">The Attempts table is a junction (realises many-to-many relationship) table, so it has a composite primary key consisting of four values (studentID, courseID, term, _year), since for each record there is only one unique combination of those values (e.g. a student can take a certain course once in a certain year's term):</div>

```
CREATE TABLE ATTEMPTS (
    studentID INT,
    courseID INT,
    _year INT NOT NULL CHECK (_year >= 2000),
    term INT NOT NULL CHECK (term IN (1,2)),
    grade INT,
    PRIMARY KEY (studentID, courseID, term, _year),
    FOREIGN KEY (studentID) REFERENCES STUDENT(studentID),
    FOREIGN KEY (courseID) REFERENCES COURSE(courseID)
);
```

<div style="text-align: left">The Prerequisite table is also a junction table, so it has a composite primary key consisting of two values (advancedCourseID,prerequisiteCourseID):</div>

```
CREATE TABLE PREREQUISITE (
    advancedCourseID INT,
    prerequisiteCourseID INT,
    PRIMARY KEY (advancedCourseID, prerequisiteCourseID),
    FOREIGN KEY (advancedCourseID) REFERENCES COURSE(courseID),
    FOREIGN KEY (prerequisiteCourseID) REFERENCES COURSE(courseID)
);
```
> 2. Write SQL-queries that insert example data into your created tables. Make sure that each table
contains at least 2 rows of data. Here are some sample data.

<div style="text-align: left">Below are attached the insert statements to fill the table up with the data given in the example:</div>

```
INSERT INTO _PROGRAM(programID, _name, requiredCPs) 
VALUES 	(1, 'Information Engineering', 120),
		(2, 'Renewable Energies', 110);
```

```
INSERT INTO STUDENT(studentID, firstName, lastName, dob, programID)
VALUES 	(123456, 'John', 'Wayne', '11-05-1998', 1),
		(234567, 'Anna', 'Meyer', '13-02-1999', 1);
```

```
INSERT INTO COURSE(courseID, _name, description, creditPoints, programID)
VALUES  (4, 'MA1', 'Mathematics 1', 8, 1),
		(9, 'MA2', 'Mathematics 2', 8, 1),
		(13, 'SS1', 'Signals and Systems 1', 6, 1),
		(15, 'DB', 'Databases 1', 6, 1);
```

```
INSERT INTO ATTEMPTS(studentID, courseID, _year, term, grade)
VALUES 	(123456, 4, 2021, 1, 7),
		(234567, 9, 2021, 2, 9),
		(234567, 13, 2021, 1, 3),
		(234567, 13, 2021, 2, 6);
```

```
INSERT INTO PREREQUISITE(advancedCourseID, prerequisiteCourseID)
VALUES 	(9, 4),
		(13, 9),
		(13, 4);
```

> 3. Write a SQL-query for the created database that returns all students (first name and last name)
that study the program “Information Engineering”.

<div style="text-align: left">The query joins two tables and retrieves the students having Information Engineering as the program:</div>

```
SELECT s.firstName, s.lastName FROM STUDENT s
JOIN _PROGRAM _p ON s.programID = _p.programID
WHERE _p._name LIKE 'Information Engineering';
```

> 4. Write a SQL-query that returns the name of all courses that have prerequisite courses.

<div style="text-align: left">The query joins two tables and only retrieves courses having matching advanced courses:</div>

```
SELECT DISTINCT _c._name FROM COURSE _c
JOIN PREREQUISITE _p ON _c.courseID = _p.advancedCourseID;
```

> 5. Write a SQL-query that returns the sum of all credit points successfully achieved by student “John
Wayne”. Keep in mind that the credit points only count when the student has an attempt with a
grade of 5 or more points.

<div style="text-align: left">The query joins three tables and sums up credit points of all passed subjects by the student:</div>

```
SELECT SUM(_c.creditPoints) FROM COURSE _c
JOIN ATTEMPTS _a ON _a.courseID = _c.courseID
JOIN STUDENT s ON s.studentID = _a.studentID
WHERE s.firstName LIKE 'John' AND s.lastName LIKE 'Wayne' AND _a.grade > 5
```

> 6. A student needs to be removed from the database. Write SQL-statements to remove the student
with the name “John Wayne” from the database.

<div style="text-align: left">The query consists of two parts. With a CTE we extract the student and, aftwerwards, delete the attempts associated with him. As a second step, we delete the student himself:</div>

```
WITH getStudent AS (
	SELECT * FROM STUDENT s
   	WHERE s.firstName LIKE 'John' AND s.lastName LIKE 'Wayne')
DELETE FROM ATTEMPTS _a
USING getStudent
WHERE _a.studentID = getStudent.studentID;

DELETE FROM STUDENT s
WHERE s.firstName LIKE 'John' AND s.lastName LIKE 'Wayne'
```

## Assignment 2

We created the tables using the script from Moodle.

> 1. Create a SQL-query that returns the dob (date of birth) of sailors in descending order that were
hired on August 3rd, 2012.

<div style="text-align: left">The query joins two tables, finds the sailors that have started working at the given day and presents their birth dates sorted in a descending order:</div>

```
SELECT s.dob FROM sailor s
JOIN hire h ON h.sailorID = s.sailorID
WHERE h.startofservice = '03-08-2012'
ORDER BY s.dob DESC;
```

> 2. Create a SQL-query that returns all information of sailors that were hired between July 3rd, 2011,
and September 3rd, 2012, and whose last name starts with a ‘J’.

<div style="text-align: left">The query joins two tables, finds the sailors satisfying the given criteria and presents all their data:</div>

```
SELECT * FROM sailor s
JOIN hire h ON h.sailorID = s.sailorID
WHERE h.startofservice < '03-09-2012' AND h.startofservice > '03-07-2011'
AND s.lastname LIKE 'J%';
```

> 3. Create a SQL-query that returns for each ship the sum of the annual salary of every sailor who
is hired for that ship.

<div style="text-align: left">The query joins three tables, finds the sum all salaries on a ship with an aggregate expression and grouping by ship:</div>

```
SELECT s.name, SUM(h.annualsalary) AS "Ship total salary" FROM ship s
JOIN hire h ON s.shipid = h.shipid
JOIN sailor sr ON h.sailorid = sr.sailorid
GROUP BY s.shipid;
```

> 4. Create a SQL-query that returns the location of all harbors that are not base harbor to any ship
in the database.

<div style="text-align: left">The query left joins two tables, so that we are left with all harbours and can easily filter out the ones that have no ships attached:</div>

```
SELECT h.location FROM harbour h
LEFT JOIN ship s ON s.baseharbour = h.harbourid
WHERE s.shipid IS NULL
```

> 5. Create a SQL-query that returns the shipId, ship name and the number of sailors who are hired
on the ship and earn maximum 42.000$.

<div style="text-align: left">The query joins two tables, groups results by ships and uses an aggregate function to count sailors with a salary below 42.000$:</div>

```
SELECT s.name, s.shipid, COUNT(h.sailorid) AS "Number of sailors" FROM ship s
JOIN hire h ON h.shipid = s.shipid
WHERE h.annualsalary <= 42000
GROUP BY s.name, s.shipid
```

> 6. Describe in your own words the result of the following query:

```
SELECT DISTINCT h1.location FROM SHIP s1, SHIP s2, HARBOUR h1, HARBOUR h2
WHERE s1.baseHarbour = h1.harbourID
AND s2.baseHarbour = h2.harbourID
AND s1.launchDate = s2.launchDate
AND h1.location = h2.location
AND h1.harbourID != h2.harbourID;
```

<div style="text-align: left">The query above fetches 4 tables and presents all possible combinations of joining records from the tables:
<br></br>
If executed line by line the result is as follows:

0) With the SELECT line executed we have 3 x 3 x 3 x 3 = 81 records in the resulting table.
1) The first condition joins s1 and h1 tables on the condition, so we are left with 3 x 3 x 3 = 27 records.
2) The second condition does the same with s2 and h2 leaving us with 3 x 3 = 9 records.
3) The third condition dictates that the resulting records should have the same launch dates which leaves us with only 3 records.
4) The fourth does nothing, since we already have the records with same locations for h1 and h2.
5) The fifth presents no data, because there are no records satisfying this condition.

