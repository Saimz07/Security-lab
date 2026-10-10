# PortSwigger Web Security Academy - SQL Injection Lab 1

**Platform:** PortSwigger Web Security Academy | **Category:** Web Security / SQL Injection | **Difficulty:** Apprentice | **Date:** 10/10/2026

## Objective

Exploit a SQL injection vulnerability in the application's `WHERE` clause to retrieve hidden data that would normally be excluded by the application's category filter.

## Approach

The application displayed products grouped by category. I focused on the category filter because selecting a category changed which products appeared on the page. This suggested that the application might use the selected category in a database query.

A vulnerable query could look conceptually like this:

```sql
SELECT * FROM products
WHERE category = 'Corporate gifts'
AND released = 1;
```

The application would use the selected category to filter products and might also exclude products that were not meant to be displayed.

I tested whether the category input could alter the SQL query's logic rather than simply select a different category.

### 1. Testing the string boundary

I first tested a single quote appended to the category value:

```text
Corporate gifts'
```

The response suggested that the input was affecting how the query was processed. This was an important clue that the category value might be inserted into the SQL statement without proper handling.

I then tested a comment-based input:

```text
Corporate gifts'--
```

This returned results. The single quote appeared to close the original SQL string, while `--` introduced a SQL comment that could cause the remainder of the query to be ignored.

This suggested that I could modify the query's logic through the category parameter.

### 2. Attempting to retrieve hidden data

After confirming that the quote-and-comment pattern affected the response, I tried adding an `OR` condition to make the category filter evaluate as true.

My first attempt was:

```text
Corporate gifts' or '1--
```

This did not work. The additional quote introduced another string boundary, leaving the expression malformed for the query structure I was testing.

My second attempt was:

```text
Corporate gifts'or 1--
```

This also failed. I was trying to make the condition true, but the expression did not use a clear comparison such as `1=1`. The exact reason an expression fails depends on the database engine and the surrounding SQL statement, so I needed to distinguish syntax problems from incorrect assumptions about how the query was constructed.

### 3. Correcting the Boolean condition

I returned to the SQL logic and used an explicit comparison:

```sql
OR 1=1
```

The intended test input was:

```text
Corporate gifts' OR 1=1--
```

The logic behind this payload was to close the original string, introduce a condition that always evaluates to true, and comment out the remaining part of the query.

Conceptually, if the vulnerable query were:

```sql
SELECT * FROM products
WHERE category = 'Corporate gifts'
AND released = 1;
```

The injected condition could transform its logic into something similar to:

```sql
SELECT * FROM products
WHERE category = 'Corporate gifts'
OR 1=1--'
AND released = 1;
```

Here, `1=1` evaluates to true. With the remaining query commented out and the SQL operator precedence taken into account, the category restriction can be bypassed, allowing products from other categories to appear.

The exact resulting query depends on how the application constructs the original statement and which SQL database it uses.

## What Was Actually Vulnerable

The vulnerability was SQL injection in the application's category-filtering query.

The application appeared to place user-controlled category input directly into an SQL statement without safely separating the data from the SQL instructions.

Instead of treating the category as a value to compare against, the database could interpret injected characters and conditions as part of the query itself.

The intended behavior was to display products matching the selected category. By manipulating the query's condition, an attacker could potentially change which rows satisfied the `WHERE` clause and retrieve products that the original filter was meant to hide.

The underlying issue was **unsafe construction of SQL queries using user-controlled input**.

## What I Actually Learned

- **A single quote can change the query's structure:** In a vulnerable query that places a category inside single quotes, an injected quote may terminate the original string and allow subsequent input to be interpreted as SQL syntax.
- **SQL comments can remove the rest of a query:** The `--` sequence starts a comment in common SQL dialects when used with the appropriate syntax. This can make the database ignore the remaining part of a statement, including additional conditions. Exact comment rules vary between database engines.
- **`OR 1=1` is a Boolean condition:** The expression compares `1` with `1`, so it evaluates to true. By contrast, `OR 1` is not universally portable SQL Boolean syntax. Some database engines accept a bare numeric value in a Boolean context, while others may reject it or handle it differently. An explicit comparison is clearer and more portable.
- **Quotes must match the surrounding SQL:** My attempt using `Corporate gifts' or '1--` failed because I introduced another quoted string without constructing a valid expression for the query's actual structure. Understanding where a string begins and ends is essential when investigating SQL injection.
- **A failed payload does not automatically mean the vulnerability is absent:** My earlier quote-and-comment test had already returned results. The later failures showed that the exact syntax and query structure mattered, rather than proving that SQL injection was impossible.
- **`WHERE` conditions control which rows are returned:** A condition such as `category = 'Corporate gifts'` restricts results to a category. An injected always-true condition can undermine that restriction if the query is vulnerable.
- **The goal is to understand the query, not memorize payloads:** Testing one change at a time helped me separate the effects of the quote, the Boolean expression, and the comment. This approach is more useful than blindly trying different strings.

## Key Takeaway

This lab helped me understand how SQL injection can turn a normal category filter into a way of retrieving hidden data. The important concept was not just adding a quote or a comment, but understanding how SQL strings, Boolean conditions, and query structure interact.

My main difficulty was constructing a valid Boolean expression after closing the original string. Reviewing the difference between `OR 1` and `OR 1=1` reinforced the importance of SQL syntax when testing injection vulnerabilities.

The main defensive lesson is to use **parameterized queries (prepared statements)** so that user input is treated as data rather than executable SQL. Input validation is useful as an additional control, but it should not replace parameterized queries.
