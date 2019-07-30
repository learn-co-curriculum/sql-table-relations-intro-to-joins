# SQL Joins

## Overview

We'll discuss how to retrieve specific sets of data from associated tables using SQL join statements. 

## Objectives

1. Describe how SQL join clauses combine data from multiple tables based on a common column between them
2. Define the different types of SQL joins: inner, outer, left outer, right outer
3. Practice writing join statements

## What Is a JOIN?

A SQL JOIN clause is a way to combine rows from two or more tables, based on a common column between them. The great thing about relational databases is that they are just that––*relational*. Relational databases allow us not only to store data that is interconnected, but to retrieve that data in ways that reflect that interconnectivity. 

Let's say, for example, we have two tables, a Cats table and an Owners table. Cats and owners are associated by a foreign key of `owner_id` in the Cats table. How would we craft a query that would grab us all of the cats with a particular owner, and even include information about that owner in the data returned to us by that query? We know how to write a `SELECT` statement that gets us all of the cats with a particular `owner_id`. For example:

```sql
SELECT * FROM cats WHERE owner_id = 2;
```

This would return us the appropriate list of cats. But what if we wanted to query *both* the Cats and the Owners tables and return information about both cats and owners? This is where JOIN statements come in. 

## JOIN Types

There are several different types of joins that we will cover in this lesson. The following JOIN keywords will be crafted into `SELECT` statements to achieve the described return values.  

| Type | Description |
|------|-------------|
| INNER JOIN | Returns all rows when there is at least one match in BOTH tables|
| LEFT [OUTER] JOIN | Returns all rows from the left table, and the matched rows from the right table|
| RIGHT JOIN* | Returns all rows from the right table, and the matched rows from the left table |
| FULL JOIN* | Returns all rows when there is a match in ONE of the tables|

`* Not supported by SQLite`

**Note:** Unfortunately, SQLite does not support the RIGHT JOIN or the FULL OUTER JOIN clauses. However, you can [emulate](http://www.sqlitetutorial.net/sqlite-full-outer-join/) the FULL OUTER JOIN by using the LEFT JOIN clause.

In the following code-along, we'll be crafting JOIN statements of each of the above types in order to return data about a series of connected database tables.

For this code-along, we'll be working with a database of pets and owners. Let's set it up:

## Setting Up the Database

* Create your database in your terminal with the following command:

```sql
sqlite3 pets_database.db
```

* Now, inside the `sqlite3>` prompt, create the following tables and insert the values:

**Cats Table:**

```sql
CREATE TABLE cats (
id INTEGER PRIMARY KEY,
name TEXT,
age INTEGER,
breed TEXT,
owner_id INTEGER);
```

**Owners Table:**

```sql
CREATE TABLE owners (id INTEGER PRIMARY KEY, name TEXT);
```

**Insert Data:**

```sql
INSERT INTO owners (name) VALUES ("mugumogu");
INSERT INTO owners (name) VALUES ("Sophie");
INSERT INTO cats (name, age, breed, owner_id) VALUES ("Maru", 3, "Scottish Fold", 1);
INSERT INTO cats (name, age, breed, owner_id) VALUES ("Hana", 3, "Tabby", 1);
INSERT INTO cats (name, age, breed, owner_id) VALUES ("Nona", 4, "Tortoiseshell", 2);
INSERT INTO cats (name, age, breed) VALUES ("Lil' Bub", 2, "perma-kitten");
```

### A Note on Foreign Keys

Note that the Cats table has an `owner_id` column. This column is a **foreign key** that connects each cat to an individual owner. If an individual cat has an `owner_id` of `2`, that indicates that that cat belongs to the owner who has an `id` of `2`. 

To confirm this, you can run the following `SELECT` statement in your `sqlite3` prompt:

```sql
SELECT * FROM cats WHERE owner_id = 2;
```

You should see just one cat returned to us, the one that belongs to Sophie, our second owner:

```sql
id               name             age         breed          owner_id        
---------------  ---------------  ----------  -------------  ----------
3                Nona             4           Tortoiseshell  2
```

 
## Code Along I: INNER JOIN

### Definition

An INNER JOIN query will return *all* the rows from both tables you are querying where a certain condition is met. In other words, INNER JOIN will select all rows from both tables as long as there is a match between the specified columns of each table.

Let's take a look at a boiler-plate INNER JOIN statement, before we try it out on our `pets_database`. 

```sql
SELECT column_name(s)
FROM first_table
INNER JOIN second_table
ON first_table.column_name = second_table.column_name;
```
This may not make sense to you just yet. Let's try it out with our own database in order to gain a better understanding. 

### Writing INNER JOINs

Enter into your database via `sqlite3 pets_database.db`, if you're not there already. Let's write an INNER JOIN query that will return the name and breed of the cat along with the name of that cat's owner.

```sql
SELECT Cats.name, Cats.breed, Owners.name 
FROM Cats 
INNER JOIN Owners
ON Cats.owner_id = Owners.id;
```

Let's break this down:
```sql
SELECT Cats.name, Cats.breed, Owners.name ...
```

Here, we are specifying which columns from each table we want to select data from. We use the `table_name.column_name` notation to grab columns from two different tables.

Next up, we join our two tables together with our `INNER JOIN` keyword: 
```sql
...FROM Cats INNER JOIN Owners
```

Lastly, we tell our query *how* to connect, or join, the two tables. In other words, we tell our query which columns in each table function as the foreign key/primary key connection. Through this, our query will correctly identify which cat belongs to which owner and return that information accordingly. 
```sql
...ON Cats.owner_id = Owners.id;
```

Here, we are telling our query that the `owner_id` column on the Cats table is filled with data that corresponds to data in the `id` column of the Owners table. We are indicating that a value of `1`, for example, in an individual cat's `owner_id` column refers to the individual owner who has an `id` of `1`. And we are telling our query to return *only* those cats and owners who share this connection.

The above statement should return the following:
```sql
name             breed            name
---------------  ---------------  ----------
Maru             Scottish Fold    mugumogu  
Hana             Tabby            mugumogu  
Nona             Tortoiseshell    Sophie  
```   

We did it! We wrote an INNER JOIN query that returns to us all of the data in the specified columns from both tables. 

Notice that the owner's name column is called `name` in the output above. That is because we requested the `name` column from the Owners table. For this particular output though, it would be great if the column could read "owners_name", to distinguish it from the cat's name column. 

Let's run that query again, this time aliasing the `name` column of the Owners table as `owners_name`, using the `AS` keyword:

```sql
SELECT Cats.name, Cats.breed, Owners.name 
AS "owner_name" 
FROM Cats 
INNER JOIN Owners 
ON Cats.owner_id = Owners.id;
```

This should return:

```
name             breed            owner_name
---------------  ---------------  ----------
Maru             Scottish Fold    mugumogu  
Hana             Tabby            mugumogu  
Nona             Tortoiseshell    Sophie 
```

### A Note on INNER JOINs, or, Where's Lil' Bub?

When we say that an INNER JOIN returns all of the data for which a certain condition is true, we mean that any data that does not meet a JOIN condition will not be returned. The JOIN condition, in this case, is the thing that our two tables are joined *on*:

`...ON Cats.owner_id = Owners.id;`

Our query, therefore, will select all of the appropriate data concerning cats and owners who *are* joined by an `owner_id`/`id` foreign key/primary key relationship. In other words, it will select all of the cats who have a value in the `owner_id` column that matches a value in the `id` column of the Owners table. Any cats that have an empty `owner_id` column, or have a value in that column that does not match the `id` of an existing owner, will not be selected by the query. 

You might have noticed that the data returned by our query did not include Lil' Bub. That's because when we inserted Lil' Bub into our Cats table, we didn't give her an `owner_id`.  

![](http://readme-pics.s3.amazonaws.com/Lil_Bub_2013_(crop_for_thumb).jpg)

Other types of JOIN statements, however, can return such data.  

## Code Along II: LEFT OUTER JOIN

I don't know about you, but I miss Lil' Bub. It would be nice if we could query our database for both cat and owner information *without* excluding her. With a LEFT OUTER JOIN we can do just that. 

### Definition

A LEFT OUTER JOIN query returns *all* rows from the left, or first, table, regardless of whether or not they met the JOIN condition. The query will also return the matched data from the right, or second, table. 

In the case of data from the first table that doesn't meet our JOIN condition, the resulting output will include `NULL`, or empty, values for the missing matched columns. 

Let's take a look at a boiler-plate LEFT OUTER JOIN:

```sql
SELECT column_name(s)
FROM first_table
LEFT [OUTER] JOIN second_table
ON first_table.column_name=second_table.column_name;
```

Now let's try it out on our `pets_database`. 

### Writing LEFT OUTER JOINs

Execute the following command in your `sqlite3>` prompt in your terminal:

```sql
SELECT Cats.name, Cats.breed, Owners.name 
FROM Cats 
LEFT OUTER JOIN Owners 
ON Cats.owner_id = Owners.id;
```

You should see the following output returned to you:

```
name             breed            name      
---------------  ---------------  ----------
Maru             Scottish Fold    mugumogu  
Hana             Tabby            mugumogu  
Nona             Tortoiseshell    Sophie 
Lil' Bub         perma-kitten                
```

Here, our LEFT OUTER JOIN has returned to us *all* of the cats (including Lil' Bub!), with matched data regarding owner's name for those cats that have an owner, and empty space in the owner's name column for the cat that doesn't have an owner. 

## RIGHT OUTER JOIN and FULL OUTER JOIN

**Important:** SQLite doesn't currently support RIGHT OUTER JOINs or FULL OUTER JOINs. However, we'll review it briefly here so you can see how it works in other Databases, like Postgres. This section isn't a code-along, just read it through and try to get comfortable with the code provided. 

### RIGHT OUTER JOIN

The RIGHT OUTER JOIN is the reverse of the LEFT OUTER JOIN. It will return *all* data from the right, or second, table and the matched data from the left, or first table. 

Let's take a look a boiler-plate RIGHT OUTER JOIN query:

```sql
SELECT column_name(s)
FROM first_table
RIGHT JOIN second_table
ON first_table.column_name = second_table.column_name;
```

Before we (pretend to) write our own RIGHT OUTER JOIN, let's insert a new owner into our Owners table:

```sql
INSERT INTO owners (name) VALUES ("Penny");
```

Now we have an owner who is not currently associated to a cat. This gives us something to work with to illustrate our RIGHT OUTER JOIN. 

### Writing RIGHT OUTER JOINs

The following query would constitute a RIGHT OUTER JOIN:

```sql
SELECT Cats.name, Cats.breed, Owners.name 
FROM Cats 
RIGHT OUTER JOIN Owners 
ON Cats.owner_id = Owners.id;
```

This would return:

```
name             breed            name      
---------------  ---------------  ----------
Maru             Scottish Fold    mugumogu  
Hana             Tabby            mugumogu  
Nona             Tortoiseshell    Sophie    
                                  Penny  
```

Notice that Lil' Bub is once again missing, but our cat-less owner, Penny, is present and accounted for. That is because the RIGHT OUTER JOIN will select *all* of the data from the second table and only the matched data from the first table. 

### FULL OUTER JOIN

FULL OUTER JOIN queries will combine the result of both a LEFT and RIGHT OUTER JOIN. In other words, they will return *all* the data from both the first and second tables.

Here's a boiler-plate example:

```sql
SELECT column_name(s)
FROM first_table
FULL OUTER JOIN second_table
ON first_table.column_name = second_table.column_name;
```

### Writing FULL OUTER JOINs

A FULL OUTER JOIN for our Cats and Owners tables would look like this:

```sql
SELECT Cats.name, Cats.breed, Owners.name
FROM Cats
FULL OUTER JOIN Owners
ON Cats.owner_id = Owners.id;
```

It would return:

```
name             breed            name      
---------------  ---------------  ----------
Maru             Scottish Fold    mugumogu  
Hana             Tabby            mugumogu  
Nona             Tortoiseshell    Sophie 
Lil' Bub         perma-kitten 
                                  Penny               
```

Our result includes both cats without owners and owners without cats. In other words, it includes *all* of our data. 

## Resources
* [SQLite FULL OUTER JOIN Emulation](http://www.sqlitetutorial.net/sqlite-full-outer-join/)

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/sql-joins-readme'>SQL JOINS</a> on Learn.co and start learning to code for free.</p>
