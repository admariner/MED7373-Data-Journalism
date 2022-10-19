# R exercises


This notebook outlines some introductory exercises to get to grips with R within RStudio, including basic data types and processes.

## Creating a variable

To work with information in R you will typically need to store it, in an object called a *variable*. In R that is done by naming your variable, and then *assigning* something to that variable using the `<-` operator, like so:

```{r}
myname <- "Paul"
```

The name of the variable above is arbitrary - we could choose anything:

```{r}
blahblah <- "Paul"
```

(The only names you can't use are '*[reserved words](https://www.programiz.com/r-programming/reserved-words)*', that have a special meaning in R, like 'if', 'else', 'function' and so on. If you choose one of these you will get a warning, so just try another word).

Once created, you should be able to see the variable in the **Environment** box in the upper right corner of RStudio. You can also now do things with that variable. You can 'call' (summon) it like this for example, like so:

```{r}
myname
```

This will 'print' the value stored in that variable, in the *Console* (a box normally in the bottom left corner of RStudio). The `[1]` at the start is just a line number.

We can also find out things about a variable, like so:

```{r}
length(myname)
nchar(myname)
```

The [`length` function](https://www.rdocumentation.org/packages/base/versions/3.4.1/topics/length) returns the length of whatever you give it. This length means the *number of items* (unlike `LEN` in Excel, which tells you the number of characters).

How do we find out the number of characters? A [quick google](https://stackoverflow.com/questions/11134812/how-to-find-the-length-of-a-string-in-r) will take you to posts suggesting the [function `nchar`](http://astrostatistics.psu.edu/su07/R/html/base/html/nchar.html).

You can find out more about a function by typing a question mark followed by the name of the function like so:

```{r}
?nchar
```

This will open the **Help** tab in the bottom right corner of RStudio at the entry for that function.

You can delete a variable by using `rm` like so:

```{r}
rm(blahblah)
```


## Different types of variable

The variable created above is a simple character object.

```{r}
typeof(myname)
```

What about variables containing numbers?

```{r}
myage = 25
typeof(myage)
```

What is a double? [Google it!](https://uc-r.github.io/integer_double/). It basically means a number with decimal points. Although 25 doesn't have any decimal points, R defaults to the double type unless told otherwise, which we can do by adding `L`:

```{r}
myage = 25L
typeof(myage)
```

Or, perhaps more easiy, by converting using `as.integer`:

```{r}
myage <- as.integer(myage)
typeof(myage)
```

Ooh! We did something there which needs a bit more explanation: changing variables. I'll come onto that in a moment. 

First, it is worth briefly mentioning some other object types that are used in R. 

* **Data tables** are objects containing a table, like a spreadsheet. 
* **Vectors** are objects containing a list of items, like a column. 

Because R is concerned with data, these are probably the two most common objects you will deal with. [More explanation of these, and other object types, can be found here](https://github.com/paulbradshaw/Rintro/blob/master/Robjects.md).

## Changing variables

Variables are useful not just to be able to call them and test them, but to *change* them. For example we might want to clean them, filter them, multiply them, and so on.

Let's say it's my birthday and I want to change my age. Here's how I might do that:

```{r}
myage <- myage+1
myage
```
The key here is to start on the right of `<-`: the line of code first performs the calculation (`myage+1`, which is 26), and then assigns the result of that calculation to the variable (`myage`). Because this is the *same* variable, it overwrites the variable which was used in the calculation.


## Creating a vector

A vector is like a column of data. It is made up of a list of entries, like so: 

`2, 5, 10`

To turn that list into a vector you need to use the [`c` function](https://stat.ethz.ch/R-manual/R-devel/library/base/html/c.html). This creates the vector, so you put the list in brackets after the `c` like so:


```{r}
myvector <- c(2, 5, 10)
#now let's call that new object
myvector
```

In the code above I've added a *comment* by starting the line with a hash symbol. The hash symbol specifies that anything on that line should not be run as code. This allows us to add an explanation for people reading the code.

Some quick tests can tell you something about the properties of a vector:

```{r}
typeof(myvector)
length(myvector)
is.vector(myvector)
class(myvector)
```

First, we see that `typeof` tells us the type of data contained *within* the vector.

Second, we see that `length` tells us the number of *items* within the vector.

We can check if something is a vector by using `is.vector`

And I've added `class` which tells us more broadly if something is numeric, for example.

You can access individual items within a vector by using an *index* in square brackets like so:

```{r}
myvector[1]
typeof(myvector[1])
```

Look closely at that `2`. See if you can spot the difference in a minute...

Now. What if there's a mix of types of item? As always, try and see what happens:

```{r}
myvector <- c(2, 5, "10")
typeof(myvector)
length(myvector)
is.vector(myvector)
class(myvector)
```

Now because *just one* item is a string, R sees the *whole vector* as a string.

Now let's grab the first item again. Can you spot the difference?

```{r}
myvector[1]
```

How about if I add another line?

```{r}
myvector[1]
typeof(myvector[1])
```

The `2` is now in quotation marks - `"2"` - because it has been *coerced* into a string, or *character object*. A vector can only contain one data type, so if your list is a mix of numbers and words, it will coerce them all to characters. To fix this you will need to filter out the non-numeric entries and format as numbers. That's for another time...

## Creating a data frame object

A data frame is basically a table. It is created using the function `data.frame()` and the ingredients for that function need to be vectors. Here's an example of the code to create a data frame and store it in a variable we're calling 'mydataframe':

```{r}
mydataframe <- data.frame(myvector)
#now let's call that new variable to see what's in it
mydataframe
```

This is a data frame with one column - it has exactly the same information as the vector. But it is stored in a different way.

Let's create another vector and use it to make a data table with two columns:

```{r}
myothervector <- c(3,6,9)
#replace the previous data frame with a new one that uses 2 vectors
mydataframe <- data.frame(myvector,myothervector)
#now let's call that new variable to see what's in it
mydataframe
```

Note that the `myvector` column has `<fctr>` (factor) underneath and `myothervector` has `<dbl>` (double). This tells us what data types are in those columns, even though both look numeric.

The names of the vector variables have been used as column names. We can access individual columns by naming them after a dollar sign like so:

```{r}
mydataframe$myvector
class(mydataframe$myvector)
class(mydataframe$myothervector)
```

This tells us that the `myvector` column is a *factor* (text), whereas the `myothervector` is a double (numbers).

You can also access columns by specifying their *index*, like so:

```{r}
mydataframe[2]
```

And you can specify individual cells by specifying their *coordinates*: an index for the row, followed by an index for the column:

```{r}
#row 3, column 2
mydataframe[3,2]
#row 2 column 1
mydataframe[2,1]
```

If you leave the column coordinate blank (but include the comma), you can access a whole row, too:

```{r}
#row 3
mydataframe[1,]
```

We can also perform calculations - but *only if the data is the right type for that calculation*. Here, for example, is what happens if you try to use the `sum` function on the `myvector` column:

```{r}
sum(mydataframe$myvector)
```

It says it is "not meaningful" for factors. In other words, `sum` expects numbers, and these have been formatted as text. But it will work for the other column:

```{r}
sum(mydataframe$myothervector)
```

Can we convert our other column? [Google it!](https://stat.ethz.ch/R-manual/R-devel/library/base/html/strtoi.html). It turns out there is a function called `strtoi` (string to integer) which can do this:

```{r}
strtoi(mydataframe$myvector)
```

Note that this will only work if all the strings can be easily converted. Words like "NA" will cause problems.

We can use this to assign the results to the same column:

```{r}
mydataframe$myvector <- strtoi(mydataframe$myvector)
class(mydataframe$myvector)
sum(mydataframe$myvector)
```

Nice.

## Bring this all together

The exercises above should help you to understand what is happening with data in R when you are dealing with it. Here's a rundown, but this time in reverse:

* A *data frame* is a type of table that you can work with in R
* Data frames are made by combining *vectors*
* A vector consists of individual *values*
* An individual value might be a *character* object (a string), or a *numeric* object like an *integer* or a *double*
* A vector can only contain one type of object. If you give it a mix of numbers and characters, it will *coerce* them all to character objects
* You can access individual entries in a vector by using *indexes* in square brackets.
* You can access individual columns in a data frame by using the name of the data frame followed by a dollar sign, followed by the column name (RStudio will show a list to help you)
* You can also access columns by specifying their index too
* And you can access cells by specifying their row-column coordinates
* You can access rows by specifying the row number as an index, followed by a comma

So far we have done this with a small vector and small data frame. This hopefully provides the foundations when you import data directly into a data frame and then manipulate it.
