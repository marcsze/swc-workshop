# Databases and SQL

## Background

There are three common options for storing data:

* Text files
* Spreadsheets
* Databases

Text files are the easiest to create and work well with version control.
We even saw how we could organize and search them in the Unix shell but we had to build the tools for even moderately complex analyses ourselves.
Spreadsheets are good for doing analysis but they don't handle large or complex datasets well.
**Databases, however, include powerful tools for search and analysis and are capable of handling large, complex datasets.**

In today's lesson, we'll be exploring the use of a database for managing data from an early twentieth-century expedition to the South Pacific.

**As with the previous lessons, I'm going to provide some brief background material and then we'll dive right into hands-on exercises.**

### Relational Databases

**The type of database we'll be working with today is called a relational database.**
Databases are arranged as tables.
**Just as in a spreadsheet,** each table has columns (also known as fields) that describe the data, and rows (also known as records) which contain the data.

To interact with the data in our database, we send commands to a database manager, which is a program that manipulates the database for us.
**This is analogous to what we saw with the Unix shell.**
Here, the database manager reads the user's command, retrieves and manipulates data from the database according to what the user asked for, and returns it for display.

### SQL

**When working with databases, the commands we type are usually called queries. The language we type them in is called Structured Query Language or S-Q-L, which can also be pronounced as "sequel."**
It's correct however you say it.

**The name Structured Query Language is a bit of a misnomer, as is the term "query," because with SQL we can do more than just query our data.**
We can add data to the database, transform existing data, combine data from different tables in powerful ways, and delete data, as necessary.

**The database manager we'll be using today is called SQLite3.**
There are many different relational database managers out there but they all speak SQL.
**While they do speak slightly different flavors or "dialects" of SQL, what you'll be learning today will help you work with any one of them.**

## Getting Started

**We'll first change some settings in SQLite3 to improve the display of our data.**

    sqlite> .mode column
    sqlite> .header on

**What do we call the word "sqlite" and with the greater-than symbol at the beginning of each line?**
This is the "prompt" in SQLite3, which performs the same role as the dollar sign in the Unix shell.

Let's perform a simple query on our data.
**We want to retrieve the names of the scientists on this expedition.**
Type exactly what you see me type here, then hit the Enter or Return key.

    sqlite> SELECT personal, family FROM Person;

**What happened? Let's go to the whiteboard for this.**

* The word `SELECT` here is the most common command we see in SQL. It tells the database manager we want to retrieve some rows.
* The word `FROM` tells the database manager which table we want to retrieve those rows from. It's followed by the name of that table, which here is `Person`.
* The words `personal` and `family` are the names of two columns in the `Person` table. They follow the `SELECT` statement, indicating that those are the only columns we want to see in the output.
* The words `SELECT` and `FROM` are capitalized here by convention; it's not required to capitalize them but it helps make your code more readable which is very important.

**Throughout today's lesson I will be typing SQL commands in all capitals and the names of columns in lowercase to help you distinguish them from the names of tables and columns in the data. I encourage you to try to do the same but you can type these commands however you want; they'll work just the same.**

    sqlite> SeLeCt PERsonal, FAMily from PERson;

**Finally, the semicolon.**
The semicolon is very import in SQL.
It tells the database manager that the command we're typing is finished.
More than once today, you might find yourself typing a command like so...

    sqlite> SELECT family FROM Person

**And you hit Enter (or Return), forgetting the semicolon!**
When that happens, you're going to see `...>` instead of the usual SQLite3 prompt.
That's okay!
SQLite3 is telling you that it's waiting for more commands.
**If you're finished with your command and you intended to type a semicolon, just type one now and hit Enter (or Return).**
SQLite3 doesn't care how many lines your command uses.

## Formatting Your Data

*e.g., wide versus narrow tables; getting figures from Chris Gates.*

## More on SELECT

**Let's get more familiar with the SELECT command.**
One thing that's neat about SQL, compared to other languages like Python, is that SQL is **declarative.**
**That means that in SQL, we tell the computer what we want to happen, not how to do it.**

For example, with the `SELECT` command, I tell the computer what columns I want to see from the Person table, **I don't tell it how to retrieve or display the data, only what I want to see.**

    sqlite> SELECT ident, family, ident FROM Person;

**In this example, note that I can ask for columns in any order and even repeat columns.**
It's important to understand that **the rows and columns in a database table aren't actually stored in any particular order.**
They will always be displayed in *some* order, but we can control that in various ways.

If I want to see all the columns of a table, a convenient shorthand is the star symbol:

    sqlite> SELECT * FROM Person;

## Sorting and Removing Duplicates

**So, let's answer some real questions about the South Pacific dataset.**
We want to know:

* What kind of quantitative measurements were taken at each site;
* Which scientists took measurements on the expedition;
* The sites where each scientist took measurements.

To answer the first question, **we can look at the Survey table to determine what measurements might have been made.**

    sqlite> SELECT quant FROM Survey;

**What's the problem with this result?**

    sqlite> SELECT DISTINCT quant FROM Survey;

**How do we determine which measurements were made at each site?**
We can use `DISTINCT` with more than one column so that every **unique pair** is returned.

    sqlite> SELECT DISTINCT taken, quant FROM Survey;

**Our next task is to identify the scientists on the expedition by looking at the `Person` table.**
Recall that the rows and columns of our tables are not stored in any particular order.
If we want to see the results in a specified order, we add the `ORDER BY` clause to our query.

    sqlite> SELECT * FROM Person ORDER BY family;

By default, results are sorted in ascending order (i.e., from least to greatest).
We can sort in the opposite order using `DESC` (for "descending"):

    sqlite> SELECT * FROM Person ORDER BY family DESC;

To be explicit about ascending order, we can add `ASC` to the query:

    sqlite> SELECT * FROM Person ORDER BY family ASC;

**In order to look at which scientist measured quantities at each site, we look again at the `Survey` table.**
We can also sort on several fields at once.

    sqlite> SELECT taken, person, quant FROM Survey ORDER BY taken ASC, person DESC;

**Looking at the table, it seems like some scientists specialized in certain kinds of measurements.**
We can examine which scientists performed which measurements by selecting the appropriate columns and removing duplicates.

    sqlite> SELECT DISTINCT quant, person FROM Survey ORDER BY quant ASC;

## Filtering

We've already answered some basic questions about our data using SQL.
**Now, we'll explore some powerful techniques for filtering our data.**

**For example, suppose we want to see when a particular site was visited.**
We can select these records from the `Visited` table by using a `WHERE` clause in our query:

    sqlite> SELECT * FROM Visited WHERE site='DR-1';

**The database manager executes this query in two stages.**
First, it checks the entires of the `Visited` table to see which ones match the `WHERE` clause.
Then, it returns the columns from those entries specified in the `SELECT` clause.
**This processing order means that we can even filter records on columns that don't appear in the results.**

    sqlite> SELECT dated FROM Visited WHERE site='DR-1';

<!--TODO Add SQL filter diagram-->

**We can combine conditional statements in a `WHERE` clause for more complex filtering.**
For example, we can ask for all information from the DR-1 site collected before 1930:

    sqlite> SELECT * FROM Visited WHERE site='DR-1' AND dated<'1930-01-01';

If we want to find out what measurements were taken by either Lake or Roerich, we can combine the tests on their names using `OR`:

    sqlite> SELECT * FROM Survey WHERE person='lake' OR person='roe';
    sqlite> SELECT * FROM Survey WHERE person IN ('lake', 'roe');

We can combine `AND` with `OR` but we need to be careful to use parentheses.
**Compare these two results. Remember to use the arrow keys to recall a previous command.**

    sqlite> SELECT * FROM Survey WHERE quant='sal' AND person='lake' OR person='roe';
    sqlite> SELECT * FROM Survey WHERE quant='sal' AND (person='lake' OR person='roe');

We can also filter by partial matches.
**For example, if we want to know something just about the site names beginning with "DR" we can use the `LIKE` keyword.**

    sqlite> SELECT * FROM Visited WHERE site LIKE 'DR%';

**Here, the `%` character is a wildcard that matches any number of characters.**

## Calculating New Values

**Returning to the expedition data... After carefully re-reading the expedition logs, we realize that the radiation measurements they report may need to be corrected upward by 5%.**
Rather than modifying the stored data, we can do this calculation on the fly as part of our query:

    sqlite> SELECT 1.05*reading FROM Survey WHERE quant='rad';

If I don't like the column header the database manager came up with, I can call it anything I want with the `AS` keyword, which also lets me rename existing columns.

    sqlite> SELECT person AS scientist, 1.05*reading AS corrected FROM Survey WHERE quant='rad';

**Note that the star character here is interpreted as a multiplication sign and not as the shortcut for all columns.**
The database manager is smart enough to tell the difference.

We also discover that Valentina Roerich's salinity measurements are entered as percentages whereas everyone else's are entered as proportions.
**Challenge: Write a query that divides Roerich's salinity measurements by 100.**

    sqlite> SELECT reading/100 FROM Survey WHERE person='roe' AND quant='sal';

This gets us the right result for Roerich's salinity measurements but **how can we get her corrected salinity measurements displayed with the other scientists' salinity measurements?**

**Let's think about this:** I want to select Roerich's corrected salinity measurements *and* I want to select the uncorrected salinity measurements from everyone else.
**How do I get salinity measurements from everyone other than Roerich?**

    sqlite> SELECT person, reading FROM Survey WHERE quant='sal' AND NOT person='roe';

Great!
The `NOT` keyword inverts my condition so that instead of selecting rows where Roerich took the measurements, it selects those rows where Roerich did *not* take the measurement.
**Now, how do we combine this query with the one that produces corrected salinity results from Roerich?**

    SELECT person, reading/100
      FROM Survey
     WHERE quant='sal' AND person='roe'
    UNION
    SELECT person, reading
      FROM Survey
     WHERE quant='sal' AND NOT person='roe';

**Several things to note about this example:**

* As mentioned earlier, I can extend my query over multiple lines to make it easier to read.
* I've used the `UNION` keyword to combine the results of two separate queries. **The results are simply stacked on top of one another. As long as they have the same columns, I can use `UNION` to concatenate the results from as many queries as I like.**
* While this may seem like a lot of work, **it's important to note that with these queries the underlying data in my database are not changed!** This is important, as any particular manipulation I do now may not be something I want to hold onto forever--I might make a mistake or I might later come to re-evaluate what I previously thought were errors.

## Missing Data

**Let's revisit this simple query.**

    sqlite> SELECT * FROM Visited;

**What's wrong with our data?**

Real-world data are never complete--there are always holes.
Sometimes, we forget to write something down for a given field or we're unable to make a certain measurement, hence, the holes in our data.
**Databases represent these holes using a special value called `null`.**
`null` is not zero, False, or the empty string; it is a one-of-a-kind value that means "nothing here."
**Dealing with `null` requires a few special tricks and some careful thinking.**

**`null` doesn't behave like other values.**

    sqlite> SELECT * FROM Visited WHERE dated<'1930-01-01';
    sqlite> SELECT * FROM Visited WHERE dated>='1930-01-01';

**These two queries should span our results but still #752 doesn't show up!**
The reason is that `null<'1930-01-01'` is neither true nor false: `null` means, "We don't know,"" and if we don't know the value on the left side of a comparison, we don't know whether the comparison is true or false.

**How can we find these holes in our data?**
This doesn't work:

    sqlite> SELECT * FROM Visited WHERE dated=NULL;

**To check whether a value is `null` or not, we must use a special test, `IS NULL`:**

    sqlite> SELECT * FROM Visited WHERE dated IS NULL;
    sqlite> SELECT * FROM Visited WHERE dated IS NOT NULL;

## Aggregation

In many scientific investigations, we want to describe the general tendency and the variation in our data.
More specifically, we often want to calculate averages and ranges for our measurements.
We can do that in databases with SQL using aggregation functions.

**We know how to select, for instance, all the dates from our measurements.**

    sqlite> SELECT dated FROM Visited;

**Getting a single value from this list of values is as simple as specifying an aggregation function.**

    sqlite> SELECT min(dated) FROM Visited;

**This gives us the earliest date. How do you think we get the latest (most recent) date?**

    sqlite> SELECT max(dated) FROM Visited;

SQL lets us do several aggregations at once.
**We can, for example, find the range of sensible salinity measurements:**

    sqlite> SELECT min(reading), max(reading) FROM Survey WHERE quant='sal' AND reading<=1.0;

**We can also combine aggregated results with raw results, although the output might surprise you:**

    sqlite> SELECT person, count(*) FROM Survey WHERE quant='sal' AND reading<=1.0;

Here, `count()` is another aggregation function; it provides a count of the number of values returned across columns.
**Why did this particular person's name appear in the query?**
The answer is that when it has to aggregate a field, but isn't told how to, the database manager chooses an actual value from the input set.
It might use the first one processed, the last one, or something else entirely.

**Here's another example.**
Suppose we suspect that there is a systematic bias in the expedition's data and that some scientists' radiation readings are higher than others.
**How would we verify this? We'd need a way of comparing mean radiation readings for each scientist.**
We know that this doesn't work because the database manager is aggregating across all scientists' readings:

    SELECT person, count(reading), avg(reading)
      FROM Survey
     WHERE quant='rad';

We could execute this query with another conditional clause five times, one for each scientist, **but that would be tedious and tedium is what we're trying to avoid by using a database!**

SQL has a `GROUP BY` clause for this very purpose.

    SELECT person, count(reading), avg(reading)
      FROM Survey
     WHERE quant='rad'
     GROUP BY person;

**`GROUP BY` does exactly what its name implies: it groups all the records with the same value for the specified field together so that aggregation can process each batch separately.**

**Just as we can sort by multiple criteria at once, we can also group by multiple criteria.**
To get the average reading by scientist and quantity measured, for example, we just add another field to the `GROUP BY` clause:

    SELECT person, quant, count(reading), avg(reading)
      FROM Survey
     GROUP BY person, quant;

Note that we have added `quant` to the list of fields displayed, since the results wouldn't make much sense otherwise.

Let's go one step further and remove all the entries where we don't know who took the measurement:

    SELECT person, quant, count(reading), avg(reading)
      FROM Survey
     WHERE person IS NOT NULL
     GROUP BY person, quant
     ORDER BY person, quant;

Looking more closely, this query:

1. Selected records from the `Survey` table where the person field was not null;
2. Grouped those records into subsets so that the `person` and `quant` values in each subset were the same;
3. Ordered those subsets first by `person`, and then within each sub-group by `quant`; and
4. Counted the number of records in each subset, calculated the average reading in each, and chose a `person` and `quant` value from each (it doesn't matter which ones, since they're all equal).

**Challenge: How many temperature readings did Frank Pabodie record, and what was their average value?**

## Combining Data

*Revisit the wide format versus narrow format tables.*

**We're about to see how to reproduce the wide format from our relational database. Along the way, we'll see one of the more powerful features of relational databases, the join.**

Let's say we're submitting our expedition data to an online mapping service that aggregates historical meteorological data.
We need to format our data as latitude, longitude, date, quantity, and reading.
However, our latitudes and longitudes are in the `Site` table, while the dates of measurements are in the `Visited` table and the readings themselves are in the `Survey` table.

**We need to combine these tables somehow. The SQL command to do this is `JOIN`.**
To see how it works, let's start by joining the `Site` and `Visited` tables:

    sqlite> SELECT * FROM Site JOIN Visited;

`JOIN` creates the cross product of two tables, i.e., **it joins each record of one table with each record of the other table to give all possible combinations.**
Since there are three records in `Site` and eight in `Visited`, the join's output has 24 records (3 times 8 equals 24).
And since each table has three fields, the output has six fields (3 plus 3 equals 6).

**What the join hasn't done is figure out if the records being joined have anything to do with each other.**
It has no way of knowing whether they do or not until we tell it how.
To do that, we add a clause specifying that we're only interested in combinations that have the same site name, thus we need to use a filter:

    sqlite> SELECT * FROM Site JOIN Visited ON Site.name=Visited.site;

**Once we add this to our query, the database manager only returns records that combined information where the site ID is the same, leaving us with just the ones we want.**
`ON` is very similar to `WHERE` and for all the queries in this particular lesson you can use them interchangeably.
This is not true in general, however.

**Notice that we used `Table.field` to specify field names in the output of the join.**
We do this because tables can have fields with the same name, and we need to be specific which ones we're talking about.
For example, if we joined the `Person` and `Visited` tables, the result would inherit a field called `ident` from each of the original tables.
**We can now use the same dotted notation to select the three columns we actually want out of our join:**

    SELECT Site.lat, Site.long, Visited.dated
      FROM Site JOIN Visited ON Site.name=Visited.site;

**We can `JOIN` as many different tables as we like.**

    SELECT Site.lat, Site.long, Visited.dated, Survey.quant, Survey.reading
      FROM Site
      JOIN Visited ON Site.name=Visited.site
      JOIN Survey ON Visited.ident=Survey.taken
     WHERE Visited.dated IS NOT NULL;

**We can tell which records from `Site`, `Visited`, and `Survey` correspond with each other because those tables contain primary keys and foreign keys.**
A primary key is a value, or combination of values, that uniquely identifies each record in a table.
A foreign key is a value (or combination of values) from one table that identifies a unique record in another table.
**Another way of saying this is that a foreign key is the primary key of one table that appears in some other table.**
In our database, `Person.ident` is the primary key in the `Person` table, while `Survey.person` is a foreign key relating the `Survey` table's entries to entries in `Person`.

Most database designers believe that every table should have a well-defined primary key.
They also believe that this key should be separate from the data itself, so that if we ever need to change the data, we only need to make one change in one place.
One easy way to do this is to create an arbitrary, unique ID for each record as we add it to the database.
This is actually very common: those IDs have names like "student numbers" and "patient numbers," and they almost always turn out to have originally been a unique record identifier in some database system or other.

**Challenge: Write a query that lists all radiation readings from the DR-1 site.**

    SELECT Survey.quant, Survey.reading, Visited.site
      FROM Survey
      JOIN Visited ON Survey.taken=Visited.ident
     WHERE Visited.site='DR-1' AND Survey.quant='rad';
