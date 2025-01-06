---
title: "SAS: merge without by, and how to fix it"
date: "2025-01-06"
---

I've been learning SAS, and a part of learning a new language is learning its footguns. One I've found relates to SAS's `merge`-construct,
specifically that you can omit the `by` statement, which leads to wildly unexpected behaviour, and even whose is completely silent. However, you
can set the 

```
OPTIONS MERGENOBY=ERROR;
```

to get SAS to report an error if the `by` statement is missing. I'm going to include that option in every SAS-script I write from now on.

## Background
In SAS, `merge` is used to perform what is known as joins in relational databases. For example, if we have two tables

```
Table A:
id | Name
---+------
1  | Alice
2  | Bob
3  | Eve

Table B:
id | Country
---+---------
1  | Austria
2  | Belgium
3  | Ethiopia
```

Then we can `merge` them together by the `id` column to get:

```
merge A B; by id;
id | Name  | Country
---+-------+---------
1  | Alice | Austria
2  | Bob   | Belgium
3  | Eve   | Ethiopia
```

That's all nice and dandy. It can even handle missing values:

```
Table C:
id | Department
---+------------
1  | Accounting
3  | Engineering

merge A C; by id;
id | Name  | Department
---+-------+------------
1  | Alice | Accounting
2  | Bob   | .
3  | Eve   | Engineering
```

Notice how SAS uses `.` to represent missing values. However, what happens if we omit the `by` statement? Catastrophe:

```
merge A C;
id | Name  | Department
---+-------+------------
1  | Alice | Accounting
3  | Bob   | Engineering
3  | Eve   | .
```
That is SO FAR from what I wanted to happen. What happens is that SAS simply runs through the rows of both tables and overwrites
the `i`'th row of the first table with the contents of the `i`'th row of the the second table.

# The solution
Since this is never what I want to in my work (at least to far), I can tell SAS to make a `merge` without a `by` an error:

```
OPTIONS MERGENOBY=ERROR;
```

Simply place this at the top of your SAS file, and this particular footgun will never shoot you again. Until you forget to add it.
