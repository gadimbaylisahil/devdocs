# SELECT

**Query steps doesn't happen in the written order**

## Concatenation

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM users;
```

```sql
SELECT first_name || ' ' || last_name AS full_name
FROM users;
```

## WHERE

*IN*  
```sql
WHERE user_name IN ('value1', 'value2', 'value3')
```

*IS NULL*  
```sql
WHERE user_name IS NULL;
```

## GROUP BY

```sql
SELECT item, COUNT(*), MAX(price)
FROM sales
GROUP BY item
```

## HAVING

HAVING is like WHERE but prior filters ROWS after grouping while latter does it prior.

With HAVING we can use aggregate functions.

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1
```

## JOINs

### Inner Join

Will join tables' rows that match on CONDITION

```sql
table1 INNER JOIN table2
  ON table1.some_column = table2.some_column
```

### LEFT JOIN

Will join tables' rows that match on CONDITION. However, left join will include rows from left side of the table even though the second table doesn't contain it.

## NULL Gotchas

```sql
SELECT * WHERE name != 'sahil'
```

Will not return rows if name is NULL

```
NULL = NULL -> this is not TRUE BUT NULL

2 = NULL -> is null

2 != NULL -> is null
```

## COALESCE

Function that returns the first argument that IS NOT NULL.

```
COALESCE(NULL, 1) -> RETURNS 1

COALESCE(name, billing_name, shipping_name) -> RETURNS whichever that has value