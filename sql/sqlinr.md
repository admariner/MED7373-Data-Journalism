# Using SQL in R with `sqldf`

You can use SQL queries in R by installing the `sqldf` package (think of it as SQL data frame). Do that first:

```{r}
install.packages("sqldf")
library(sqldf)
```

Next we need some data to run our query on. In the same folder as this project you will find some data on MPs' expenses. Import that:

```{r}
exes1718 <- read.csv("https://raw.githubusercontent.com/paulbradshaw/MED7373-Data-Journalism/master/sql/exes1718.csv", stringsAsFactors = F)
```

## The first query: SELECT

Now for a basic query. To use a SQL query in R you just use the `sqldf` function and put your SQL query inside the brackets that follow. Note that the query is enclosed in quotation marks (either single or double will work), so if you want to specify a string in your query, you need to make sure that string uses a different type of quotation mark.

All SQL queries need to include the `SELECT` command to indicate which fields you want to use in your query; and the `FROM` command to indicate what table those fields are being taken from.

The most basic SQL query, then, looks like this:

```{r}
sqldf('SELECT *
      FROM exes1718')
```

The `*` means 'all fields', so this query simply means 'select everything from the table 'exes1718'.

We can instead specify only those fields we want to see, like so:

```{r}
sqldf('SELECT Date
      FROM exes1718')
```

If we want more than one field, we separate each with a comma, like so:

```{r}
sqldf('SELECT Date, Year
      FROM exes1718')
```

In the query above we have put the commands in capital letters, and each part on a different line, but that isn't essential - it just makes it easier to read.

You could just as easily write it like this:

```{r}
sqldf('select Date, Year from exes1718')
```

As you can see, writing like that, however, does make it harder to distinguish between commands (like SELECT and FROM) and ingredients (the names of tables or fields).

### Selecting columns with full stops

When R imports data it replaces spaces with periods...

```{r}
colnames(exes1718)
```

...But this can be problematic for SQL queries.

Note how SQL struggles with column names containing periods:

```{r}
mpnamesonly <- sqldf("SELECT Journey.Type, Amount.Paid
                     FROM exes1718")
```

The reason for this is that in a SQL query a period normally indicates a relationship with a table: to describe the field 'name' in a table called 'people' you might use `people.name`. So when it sees `Journey.Type` it actually looks for a table called 'Journey' which it assumes has a field called 'Type' in it.

Because there's no such table, it generates an error.

One solution is to put any names containing periods in ``` marks like so:

```{r}
journeytypesonly <- sqldf("SELECT `Journey.Type`, `Amount.Paid`
                     FROM exes1718")
```

Or you may prefer to rename them first:

```{r}
#Show the column names
colnames(exes1718)

#Now rename some of the column names
colnames(exes1718)[4] <- "mpname"
colnames(exes1718)[5] <- "mpconstituency"
colnames(exes1718)[7] <- "expensetype"
colnames(exes1718)[8] <- "description"
colnames(exes1718)[16] <- "amountclaimed"
colnames(exes1718)[17] <- "amountpaid"
#And check it
colnames(exes1718)
```

A quicker way is to use `gsub` to replace full stops. Because this uses **regex** you need to escape the full stop in square brackets like so: `[.]` so that it interprets it literally rather than to mean 'any character'

```{r}
#Show the results of gsub:
gsub("[.]","_",colnames(exes1718))
#Now assign those as column names:
colnames(exes1718) <- gsub("[.]","_",colnames(exes1718))
colnames(exes1718)
```


## Adding a filter: `WHERE`

Selecting everything from a table or its columns isn't going to be much use. So let's add another command: `WHERE`. This will filter the dataset based on a criteria we specify:

```{r}
sqldf('SELECT Date, Year, amountclaimed
      FROM exes1718
      WHERE amountclaimed = 1000')
```

If the criteria is mathematical (greater than, less than, equals, etc.) then we can simply use an operator like `>`, `<`, `=`. (You can also use `!=` to indicate 'not equal to').

You can specify numbers within a range using `BETWEEN`:

```{r}
sqldf('SELECT Date, Year, amountclaimed
      FROM exes1718
      WHERE amountclaimed BETWEEN 1000 AND 2000')
```

If your query relates to text, you can use the command `IS`, like so:

```{r}
sqldf('SELECT Date, mpname, amountclaimed, Category
      FROM exes1718
      WHERE Category IS "Accommodation"')
```

These queries show the results, but let's store them instead, in a new object:

```{r}
accommclaims <- sqldf('SELECT Date, mpname, amountclaimed, Category
      FROM exes1718
      WHERE Category IS "Accommodation"')
#Show the first few rows
head(accommclaims)
```

Of course the same result can be achieved by using base R's own `subset` function, but you may find this more intuitive.

### String queries: `LIKE`

What if we want to grab partial matches where a word is somewhere in that column? The `LIKE` command used with the `%` operator as a wildcard allows us to do that:

```{r}
sqldf('SELECT Date, mpname, amountclaimed, Category
      FROM exes1718
      WHERE mpname LIKE "%Smith%"')
```

### Using `OR` and `AND` to broaden or narrow the filter

We can broaden the query further by adding `OR` like so:

```{r}
sqldf('SELECT Date, mpname, amountclaimed, Category
      FROM exes1718
      WHERE mpname LIKE "%Smith%"
      OR mpname LIKE "%Jones%"')
```

Note that *after* `OR` you need to repeat the name of the column and `LIKE` again.

Alternatively, we can use `AND` to specify multiple criteria:


```{r}
sqldf('SELECT Date, mpname, amountclaimed, Category
      FROM exes1718
      WHERE mpname LIKE "%Smith%"
      AND Category LIKE "%Office%"')
```


## Calculations using SQL: `COUNT`

SQL allows us to perform calcuations on the data too, such as counting matches and calculating totals, averages, and so on...

Here are 2 SQL queries asking to count how many entries in the data frame are above 100, and how many are below 100.

```{r}
sqldf('SELECT COUNT(*)
      FROM exes1718
      WHERE amountpaid > 100')
sqldf('SELECT COUNT(*)
      FROM exes1718
      WHERE amountpaid < 100')
```
How would you adapt that query to ask how many are exactly 100? Or above 1000?

### Calculations: `SUM`

To add them, just use `SUM` instead of `COUNT` - but this time you need to make sure you are using `SUM` with a numerical column:

```{r}
sqldf('SELECT SUM(amountpaid)
      FROM exes1718
      WHERE amountpaid >100')
```

Note that the column you are testing, and the column you are adding, do *not* have to be the same. We can write another query that adds up the amount paid where the category is accommodation:


```{r}
sqldf('SELECT SUM(amountpaid)
      FROM exes1718
      WHERE category IS "Accommodation"')
```


### Calculations: averages (`AVG`)

Besides counts and sums, we can also calculate averages using `AVG` for the mean:

```{r}
sqldf('SELECT AVG(amountpaid)
      FROM exes1718
      WHERE category IS "Accommodation"')
```

Or `MEDIAN` for the middlemost value:

```{r}
sqldf('SELECT MEDIAN(amountpaid)
      FROM exes1718
      WHERE category IS "Accommodation"')
```

...and the `MODE` (most commonly occurring number):

```{r}
sqldf('SELECT MODE(amountpaid)
      FROM exes1718
      WHERE category IS "Accommodation"')
```

### Calculating biggest and smallest amounts

Not surprisingly, `MAX` and `MIN` can be used to find those values. Here we switch to a different criteria too:

```{r}
sqldf('SELECT MAX(amountpaid)
      FROM exes1718
      WHERE mpname IS "Adrian Bailey"')
```

And for minimum:

```{r}
sqldf('SELECT MIN(amountpaid)
      FROM exes1718
      WHERE mpname IS "Adrian Bailey"')
```


### Statistical queries

We can also ask for things like the standard deviation, an indication of how widely numbers vary:

```{r}
sqldf('SELECT STDEV(amountpaid)
      FROM exes1718
      WHERE description IS "Office Costs"')
```

## Grouping results - pivot tables using `GROUP BY`

Of course we probably don't want to have to do this for every category or MP. So the `GROUP BY` command in SQL is useful to perform a calculation for all categories, places or people with a breakdown for each:


```{r}
sqldf('SELECT SUM(amountpaid)
      FROM exes1718
      GROUP BY category')
```

Note that the categories themselves in the above query aren't shown. Why? Because we haven't selected them! Let's rectify that:

```{r}
sqldf('SELECT category, SUM(amountpaid)
      FROM exes1718
      GROUP BY category')
```

You can even add other calculations to make a bigger table:

```{r}
sqldf('SELECT category, STDEV(amountpaid), MAX(amountpaid), MIN(amountpaid), AVG(amountpaid), MEDIAN(amountpaid), SUM(amountpaid)
      FROM exes1718
      GROUP BY category')
```

It is interesting that staffing seems to have the widest variation even though its maximum and minimum numbers are not unusually large or small.

### Sorting results: `ORDER BY`

So it looks like staffing costs vary most. But we can make it easier by adding an `ORDER BY` command:

```{r}
sqldf('SELECT category, STDEV(amountpaid) FROM exes1718
      GROUP BY category
      ORDER BY stdev(amountpaid)')
```

By default, the results will be ordered *ascending*, from smallest to largest. To specify that you want it to be ordered from largest to smallest (or from A to Z with text), add `DESC` to the end of that line:

```{r}
sqldf('SELECT category, STDEV(amountpaid) FROM exes1718
      GROUP BY category
      ORDER BY stdev(amountpaid) DESC')
```



## Saving and exporting results

Of course any results can be stored in an R object and/or exported like so:

```{r}
# How to create a new object
exesstdevbycategory <- sqldf('select category, stdev(amountpaid) from exes1718 group by category')
# How to write results as a CSV
write.csv(sqldf('select category, stdev(amountpaid) from exes1718 group by category'), "exesbycategorystdev.csv")
```


## Joining tables

So far we have queried just one table, but one of SQL's strengths is that it allows you to perform queries *across* **multiple tables**. So, for example, if the data about what party each MP belongs to was in a separate table, we could include that data in our SQL query of the separate table on how much they claimed.

To do that we need to use one of the `JOIN` commands in SQL.

This is best demonstrated with two simple tables, that we will create ourselves:

```{r}
#Create two vectors
names <- c("Paul","Jane","Mo")
ages <- c(18,24,66)
#Create a data frame with those as columns
people <- data.frame(names, ages)
#Check it worked
people

#Repeat for a second data frame
names2 <- c("Claire", "Jane", "Paul")
birthplace <- c("Birmingham","Birmingham","Bolton")
places <- data.frame(birthplace,names2)
places
```

Both tables have data in common: the names of some people. However, there are some differences, too: 'Mo' only appears in one table and 'Claire' only appears in the other.

This will be important when we join them.

The two main joins used are `INNER JOIN` and `LEFT JOIN`. Here's what happens with `INNER JOIN`:

```{r}
#INNER JOIN the people table to the places table
sqldf::sqldf("SELECT *
             FROM people
             INNER JOIN places ON places.names2 = people.names")
```

Notice that 'Mo' doesnt appear in the results at all; neither does 'Claire'. That's because an `INNER JOIN` only returns results where the matching data (in this case names) appears in *both* tables.

Now for the `LEFT JOIN`:

```{r}
#LEFT JOIN the people table to the places table
sqldf::sqldf("SELECT *
             FROM people
             LEFT JOIN places ON places.names2 = people.names")
```

This time 'Mo' does appear, while 'Claire' still doesn't. This is because a `LEFT JOIN` will retain *all* data from the first table (the one 'on the left') even if there's no matching data for it in the other table.

However, it still drops data from the other table if it doesn't match - so 'Claire' is left out.

A `RIGHT JOIN` would do the opposite - 'Claire' would remain in the data (because she is in the second, 'right hand', table) while 'Mo' wouldn't. However, `sqldf` does not allow you to perform a `RIGHT JOIN` (other SQL tools do, however).

Now, to explain the syntax behind the query. Let's look at it again:

```{r}
sqldf("SELECT *
      FROM people
      LEFT JOIN places
      ON places.names2 = people.names")
```

The first part is normal: `SELECT *` selects all the fields.

Secondly, you specify just *one* of the two tables that you want to join: this should be the one that you want to keep all the rows for (the 'left' one):

`FROM people`

The second table is named in the 'JOIN' part:

`LEFT JOIN places`

And finally we specify what we are joining *on* - in other words, which two columns are we matching:

`ON places.names2 = people.names`

Notice that at this point we have to specify both the table name *and* the field name, with a period joining them together. We also use `=` to indicate the connection.

We *can* get away with not naming the tables if all the fields are different, but it's best to get into the habit of using the table.field approach because often fields *do* have the same names.

### Expanding JOIN queries to include filters and calculations

Once we have joined tables the results can form part of a bigger query across those tables like so, adding extra commands like `WHERE` etc.:

```{r}
#LEFT JOIN the people table to the places table
sqldf("SELECT *
      FROM people
      LEFT JOIN places ON places.names2 = people.names
      WHERE places.birthplace IS 'Bolton'")
```

Note that when doing this, it's a good idea to continue using the table.field naming convention outlined earlier, to avoid any problems when referring to a field which could come from either table.

## Extra: showing a summary of categories: `SELECT DISTINCT`

You can add `DISTINCT` to the `SELECT` command to only show distinct entries from that field. For example:

```{r}
sqldf("SELECT DISTINCT Journey_Type
      FROM exes1718")
```

We get a list of the 10 distinct values in that column.

If we specify more than one column, the command will show distinct *combinations*. For example:

```{r}
sqldf("SELECT DISTINCT Journey_Type, Status
      FROM exes1718")
```

Now we have 15 distinct values: of the 10 shown previously some only have one combination with 'Status' (normally 'Paid'), but some have more than one.

'Between London & Constituency', for example, has 3 distinct combinations: with 'Paid', 'Not Paid' and 'Repaid'. Likewise blank entries in 'Journey_Type' also have the same 3 distinct combinations. 'Within Constituency Travel' has two distinct combinations.
