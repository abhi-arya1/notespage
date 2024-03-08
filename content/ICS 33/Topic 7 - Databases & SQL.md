---
title: Topic 7 - Databases and SQL
date: 2024-03-08
---

> We want to set up a way to communicate with other programming languages to solve problems. How?

- We have to work with data, and use Python data structures as _tools_ to write those realistically sized programs.
- We can build a _DBMS_, or <u>Database Management System</u> to do so.
- Now, we can also analyze the costs of those choices in our software architecture, both for $O$ notation of memory and runtime.

> Relational Databases

- A relational database is one where we store data elements that can be explicitly related to one another.
- By making those relationships for, say, a database of students/classes/systems at UCI, we can ask questions like:

  - "How many students were in courses under Professor `____` during the `____--____` academic year"

- Let's define some more terms for a Relational Database

- A relational database is made up of _tables_, which consist of _rows_ and _columns_
- Each column corresponds to a unique "name" in a row is made up of a value that has a known type, such as _str_. The DBMS will enforce this strongly, unlike python's loosely typed style.

From Thornton:

```
Your background in Python offers a reasonably good mental model for how a table might be arranged if it was stored in memory by a Python program: as a list containing tuples, where the list is not sorted in a particular order, and where every tuple has the same number of values in the same relative order.

A list turns out to be a good approximation, because it offers the core understanding that we can find a row in a table most quickly if we know where it's stored, just as we can find an element in a Python list most quickly if we know its index.

Each data element being a tuple is a good approximation, too, because each row is made up of values in multiple columns, but we know their structure ahead of time — `course_number` is always first, followed by `course_name`, and then `unit_count` — so we can quickly obtain exactly the values we want by knowing that structure, just like we can when we use tuples in Python.

There are a couple of additional small issues with a surprisingly big impact if we ignore them, so it's best for us to nail them down now.

- We generally want every row in a table to be distinguished from the others in some way. Even if there are two rows with the same value in one column, there should never be two rows with the same values in all columns, so we can be sure that there's always a way to distinguish two rows from each other.
-
- We generally want each column to be made up of a value you might call a _scalar_, which is to say that there's no advantage if we break it into smaller values, since they wouldn't be individually or separately meaningful. Reasonable people can disagree on where to draw the line, but that's the general goal to bear in mind.

Neither of these properties will necessarily be enforced automatically, but we'll nonetheless want to keep them in mind when we decide how to organize our information into tables, a topic we'll consider in a little more depth shortly.
```

- i.e. Tables are Lists, Rows are Tuples, Rows are distinguishable from one another.
- You shouldn't be able to break up an element within a column into smaller values. i.e. its a _scalar_ and has reached its minimum state (as opposed to Vectors or Matrices with many values)

#### Primary Keys

- While we are talking about these, let's consider the fact that there should _always_ be a way to distinguish between two courses.
  - This is known as a **_Primary Key_**.
    - In the example of courses at UCI, this could be a course code.
- Locating something by its columns is called an _index_, and is the conventional way to go about this.
- Remember, Primary Keys are Unique and _STABLE_ (i.e. Unchanging)

#### Relationships

- We can consider different types of relationships between tables within the same relational database.
- Considering tables `A` and `B` in the same DB:
  - _One to One_ --> One element of B can be related to One element of A, and vice versa
    - If we had a table specifying identifying information for students and another table containing students' UCInetIDs and their associated login credentials (password, etc.), each student would be associated with one UCInetID and one UCInetID could only be associated with one student.
  - _One to Many_ --> One Element of A Relates to Many in B, but not vice versa.
    - If we had a table specifying available *time slots* (i.e., a block of time on a particular day of the week during which a course can be scheduled to meet in one classroom), each time slot can only accommodate one course, but one course can occupy multiple time slots (e.g., by meeting on both Tuesdays and Thursdays).
  - _Many to Many_ --> Many different elements in A relate to many in B
    - This is certainly the nature of the enrollment relationship between students and courses, since students can enroll in many different courses, and the same course can have many different students enrolled in it.
  - But remember, we don't want to always be flexible, but rather flexible when they need to be.

#### Foreign Keys

- Lets consider these two tables:

| Student    |            |           |
| ---------- | ---------- | --------- |
| Student ID | First_Name | Last_Name |
| 123        | Boo        | Thornton  |
| 234        | Some       | Person    |

| Student   | Login      |
| --------- | ---------- |
| UCI NetID | Student ID |
| boo       | 123        |
| sperson   | 234        |

- The primary key of the `student` table is `student_id`.
- The primary key of the `student_login` table is `ucinetid`.
- The `student_id` column in the `student_login` table is a foreign key referring to the `student_id` column of the `student` table (i.e., uniquely identifying `student` rows by their primary key).

- With this system, we can enforce something called _referential integrity_. i.e. if we store a `student_login` for a nonexistent `student_id`, that won't be allowed, and if we try to update a row from `student` where there is a corresponding `student_login`, we can have it fail.

- We can also make `student_id` unique in the Login table, or we could make a _one to many_ or _many to many_ relationship in this case too.
  - A one-to-many relationship could be implemented similarly, with the table on the "many" side of the relationship having a foreign key referring back to a row in the table on the "one" side, but without the restriction that the foreign key be unique in the "many" table.

![[Screenshot 2024-01-25 at 5.57.03 PM.png]]

- Let's break this down by table:
- In the `student` table:
  - `student_id` is the primary key, and others are the data relating to it
- In the `course` table:
  - `course_number` is the primary key, and others are the data relating to those
- In `enrollment`:

  - `student_id` and `course_number` are both _foreign_ keys, but this table essentially relates two _primary_ keys in different tables _to_ each other.

- Implementing all of this is _not at all_ easy to do by hand. It requires a deep study of this field and we can't do anything but bow down as mere mortals and use a **standard library package** 😳

- We can do this with the package called _sqlite_, an embedded database system, where we run the entire database as a program as a library.
- We can no longer talk across a network, and lots of programs to communicate simultaneously, but for the use cases of this class it is ok.

### SQLite3

- SQLite3 falls under the SQL family-->PostgreSQL, MySQL, MariaDB, etc.
- SQL is a **_Structured Query Language_**

  - A way to query for databases, and it is a rather powerful tool.

- How can we create that table in SQL then? (All Caps in SQL are keywords)

```sql
CREATE TABLE Person(
	person_id INTEGER PRIMARY KEY,
	name TEXT,
	age INTEGER
) STRICT;
```

What does this mean exactly?

- We are creating a table named `Person`

  - Each row in `Person` will have a:
    - `person_id`, an Integer Primary Key
    - `name`, some Text
    - `age`, an Integer
  - and `STRICT` means that we _want_ SQLite to type-check what is in the columns. This is not the default, but it's what we want, and this is how we get it.

- How can we navigate around this database?
  - We can use the `cursor` object to navigate the connection.

````python
>>> import sqlite3
>>> connection = sqlite3.connect(':memory:')

>>> cursor = connection.execute('SELECT name FROM sqlite_schema;')
# This query asks for the names of all tables that we've created.
# The sqlite_schema table is where metadata about the database's
# tables is stored.

```python-shell
>>> connection.execute(
...     '''
...     CREATE TABLE person(
...         person_id INTEGER PRIMARY KEY,
...         name TEXT,
...         age INTEGER
...     ) STRICT;
...     ''')

>>> <sqlite3.Cursor object at 0x000001EC2EA93F40>

>>> cursor.fetchone()
    ('person',)
# Each call to fetchone on a cursor returns one row of our
# result, expressed as a tuple whose values correspond to the
# columns we've asked for.  We've selected one column called
# name, so we're seeing one-element tuples returned to us.

>>> cursor.fetchone()
# fetchone returns None when there are no more rows.

>>> cursor.close()
# We can (and should) close cursors when we're done with them.
````

The problem is, we want to write SQL code, but we can't without some issues. To learn and write SQL, use the provided SQLite Shell found here: `/Users/ashwa/Documents/UCI/Winter_24/ICS33/Other/SQLite3_Shell.py`

Here is a general overview for how to manipulate databases in SQL. Note that all examples will be ran in the configuration with the `person` table above:

### A Syntactic Overview of SQLite

```sql
-- assuming this table
CREATE TABLE person(
	person_id INTEGER PRIMARY KEY,
	name TEXT,
	age INTEGER
) STRICT;

-- (This is a comment in SQL)
/* (so is this) */

-- How can we insert a new person into the table then?
INSERT INTO person (person_id, name, age)
VALUES (1, 'Boo', 13);

-- now, inserting a new user with the `person_id` of 1 would raise an error
-- since that is our *primary key*
-- we will also be checking types properly, since we mentioned that
-- in the `person` declaration (INTEGER -> int, TEXT -> str)

-- Let's insert another person
INSERT INTO person (person_id, name, age)
VALUES (2, 'Alex', 49)

-- Lets get all those back
SELECT name
FROM person
>>> ('Alex',)
>>> ('Boo', )
```

- Now, how can we get the data that we are looking for/that we need?

```sql
-- Now, let's *query* (or *select*) from the database what we need:
SELECT name
FROM person
WHERE person_id = 1;
>>> ('Boo',)
/* Note that we could always smash this all onto one line, but we don't want to do that because we are writing for *people*, not *programs* */

-- What about multiple checks and functions?
SELECT name
FROM person
WHERE length(name) = 4
AND
person_id BETWEEN 1 AND 10;
>>> ('Alex',)

-- What about Truthiness?
SELECT name
FROM person
WHERE age;
>>> ('Boo',)
>>> ('Alex',)
/* This is if we wanted to check the truthiness of AGE,  If we wanted to be strictly clear, we can do the neq operator below */
SELECT name
FROM person
WHERE age <> 0;
>>> ('Boo',)
>>> ('Alex',)
-- <> is the Not-Equal (neq) Operator, instead of just "="
```

- What if we want to control the sequencing of rows?

```sql
-- Remember that *selection* in SQL happens at the very end
SELECT name
FROM person
ORDER BY age ASC;
>>> ('Boo',)
>>> ('Alex',)
-- While "ORDER BY age DESC" would give:
>>> ('Alex',)
>>> ('Boo',)

-- We can also get the order from the query itself:
SELECT name
FROM person
WHERE age < 40
ORDER BY age ASC;
>>> ('Boo',)
/* These statements MUST be listed in this order, which is *mostly* logical, but it just requires straight memorization. SQL syntax is weird */

-- There's even a LIMIT clause, which doesn't give you more rows than given
SELECT name
FROM person
ORDER BY age ASC
LIMIT 1;
>>> ('Boo',)
```

- Now how do I change rows that I've already been given?

```sql
-- Let's use the UPDATE operation
UPDATE person -- specify the table to update
SET age = age + 1, -- set with: [right = row as was, left = row as needed]
	name = 'Test' -- we can chain SET statements
WHERE person_id = 2 -- we can update specific entries

/* Note that messing up a SET without a where *can and will* overwrite THE ENTIRE TABLE. Be careful with that. */

-- How to delete?
DELETE
FROM person
WHERE person_id = 3;
/* if we don't have a person with id 3, it won't do anything */

-- If this happens:
DELETE
FROM person;
-- this will delete the entire table

-- We can also delete an *ENTIRE* table with a certain operation:
DROP TABLE person;
-- This drops the person table from the database.
```

- Now that that table is deleted, lets get more specific and enforce data integrity

```sql
CREATE TABLE course(
	course_id INTEGER PRIMARY KEY,
	course_number TEXT,
	title TEXT,
	unit_count INTEGER
)

-- When we put this in, the (title, unit_count) == (None, None)
INSERT INTO course (course_id, course_number)
VALUES (1, 'ICS 31')
-- We get (None, None) bc of Python, but in SQL, this is called "NULL"

-- Sidenote, the * operator:
SELECT *
FROM course:
>>> (1, 'ICS 31', None, None)
-- This gets all columns from all rows in the table

-- Lets check for NULL data then:
SELECT *
FROM course
WHERE title IS NULL: -- Do this instead of "= NULL" otherwise you won't get it back
>>> (1, 'ICS 31', None, None)
/* NULL = NULL is False in SQL */

-- Goodbye, World
DROP TABLE course
```

- What if we have to _enforce_ that data _cannot_ be unknown?

```sql
-- Let's add *constraints* on the Data
CREATE TABLE course(
	course_id INTEGER NOT NULL PRIMARY KEY, -- Note that NOT NULL is true of PRIMARY KEY cols anyways
	course_number TEXT NOT NULL UNIQUE,
	title TEXT,
	unit_count INTEGER NOT NULL CHECK (unit_count > 0)
) STRICT;

-- Now that we made a safety net,
INSERT INTO course (course_id)
VALUES (1);
>>> Error: NOT NULL Constraint Failed

/* We can also put UNIQUE like "title TEXT UNIQUE" to make the titles always unique, and we can also put a CHECK like in `unit_count` to make sure our data is satisfactory */
-- Now giving a repeat title or a nonpositive unit count would give errors

-- This helps our data become production-ready and professional

-- We can also add a row-check:
DROP TABLE course
CREATE TABLE course(
	course_id INTEGER NOT NULL PRIMARY KEY,
	course_number TEXT NOT NULL UNIQUE,
	title TEXT,
	min_unit_count INTEGER NOT NULL CHECK (unit_count > 0)
	max_unit_count INTEGER NOT NULL,
	CHECK (max_unit_count >= min_unit_count)
) STRICT;
-- This is another way to check stuff that could go wrong!
```

- Now, remember relational databases? How can we implement those here?
- Let's set that up with more table specification:

```sql
CREATE TABLE student(​
	student_id INTEGER NOT NULL PRIMARY KEY,​
	name TEXT NOT NULL CHECK (length(name) > 0)​
) STRICT;
 ​
CREATE TABLE course(​
	course_id INTEGER NOT NULL PRIMARY KEY,​
    course_number TEXT NOT NULL CHECK (length(course_number) > 0),​
    unit_count INTEGER NOT NULL CHECK (unit_count > 0)​
) STRICT;
 ​
CREATE TABLE enrollment(​
	student_id INTEGER NOT NULL,​
	course_id INTEGER NOT NULL,​
    PRIMARY KEY (student_id, course_id),​
    FOREIGN KEY (student_id) REFERENCES student(student_id),​
	FOREIGN KEY (course_id) REFERENCES course(course_id)​
) STRICT;​

-- This is a system of *linkages* between tables, so tables with columns with values
-- can express that type of relationship

/* assuming that we put everything in that can be found in the notes:
https://ics.uci.edu/~thornton/ics33/Notes/Databases/
-- let's make those relationships */

-- We need to *find* the data that's available to us, by JOINing
SELECT s.name
FROM student as s
	INNER JOIN enrollment AS e ON e.student_id = s.student_id
	INNER JOIN course AS c ON c.course_id = e.course_id
	-- now we set up those relationships to the object!
WHERE c.course_number = 'ICS 33';
>>> ('Boo',) -- now we get the correct output!
-- We joined together the enrollment and course tables and matched them
-- to the student table.
```
