---
layout: post
classes: wide
title:  JSON improvements in SQLite 3.38.0
date:   2022-02-27 00:00:29 +0530
categories: programming
---

SQLite [3.38.0](https://www.sqlite.org/releaselog/3_38_0.html) introduced improvements to JSON query syntax using `->` and `->>` operators that are similar to [PostgreSQL JSON functions](https://www.postgresql.org/docs/9.5/functions-json.html). In this post we will look into how this simplifies the query syntax.

### Installation

> The JSON functions are now built-ins. It is no longer necessary to use the -DSQLITE_ENABLE_JSON1 compile-time option to enable JSON support. JSON is on by default. Disable the JSON interface using the new -DSQLITE_OMIT_JSON compile-time option. 

With the release of 3.38.0 JSON support is on by default. SQLite also provides pre-built binaries for Linux, Windows and Mac. For Linux you can get started with below

```shell
wget https://www.sqlite.org/2022/sqlite-tools-linux-x86-3380000.zip
unzip sqlite-tools-linux-x86-3380000.zip
cd sqlite-tools-linux-x86-3380000
rlwrap ./sqlite3
```

I found that support for readline is missing in the prebuilt binaries. If readline and other build utilities are installed in your machine you can download and compile SQLite binary.

```shell
wget https://www.sqlite.org/2022/sqlite-autoconf-3380000.tar.gz
tar -xvf sqlite-autoconf-3380000.tar.gz
cd sqlite-autoconf-3380000
./configure
make
./sqlite3
```

### Sample data

In this post we will be using a sample data that stores the interests of a user as a JSON and see how the new operators provide support in querying. The table has an auto incrementing id as primary key with name as text and a JSON storing interests of the user. Let's also assume the array of likes are stored in the order of preferences such that for John he likes skating the most and swimming as the last.

```sql
./sqlite3
SQLite version 3.38.0 2022-02-22 18:58:40
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> create table user(id integer primary key, name text, interests json);
sqlite> insert into user values(null, "John", '{"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]}');
sqlite> insert into user values(null, "Kate", '{"likes": ["reading", "swimming"], "dislikes": ["skating"]}');
sqlite> insert into user values(null, "Jim", '{"likes": ["reading", "swimming"], "dislikes": ["cooking"]}');
sqlite> .mode column
sqlite> select * from user;
id  name  interests                                                   
--  ----  ------------------------------------------------------------
1   John  {"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]}                                               
2   Kate  {"likes": ["reading", "swimming"], "dislikes": ["skating"]}
3   Jim   {"likes": ["reading", "swimming"], "dislikes": ["cooking"]}
```

### Operator semantics

[SQLite docs](https://www.sqlite.org/json1.html#jptr)

With the above schema in place we can now use the JSON operators to query. With `->` we can have the left part as a JSON component and the right part as a path expression. `->` also provides a JSON representation in return so it can be chained in nested lookups. In the below example we look for users who have reading as their first preference. Since the interests column is of JSON type we can use `->` operator along with `$.likes` which is a path expression to get the likes attribute. Then we can pass it to `->>` which also takes a path expression to its right but returns value as SQLite datatype like text, integer etc. instead of JSON. Here `$[0]` is used to access the first element of the array. As per docs path also supports negative indexing where `$[#-1]` can be used to access last element of the array.

```sql
select id, name, interests from user 
where interests->'$.likes'->>'$[0]' = 'reading';

id  name  interests                                                  
--  ----  -----------------------------------------------------------
2   Kate  {"likes": ["reading", "swimming"], "dislikes": ["cooking"]}
3   Jim   {"likes": ["reading", "swimming"], "dislikes": ["cooking"]}

select id, name, interests from user 
where interests->'$.likes'->>'$[0]' = 'skating';

id  name  interests                                                   
--  ----  ------------------------------------------------------------
1   John  {"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]}
```

The above query can be further simplified removing `$`

```sql

select id, name, interests from user 
where interests->'likes'->>'[0]' = 'skating';

id  name  interests                                                   
--  ----  ------------------------------------------------------------
1   John  {"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]}
```

The catch with `->` and `->>` is that `->` returns a JSON representation and it also expects the right side to be a JSON. This might lead to some confusion like below example where using `->[0]` doesn't return any value since we are comparing JSON representation with text value of "skating"

```sql
-- Returns no value

select id, name, interests from user 
where interests->'likes'->'[0]' = 'skating';

-- Returns value

select id, name, interests from user 
where interests->'likes'->'[0]' = '"skating"';

id  name  interests                                                   
--  ----  ------------------------------------------------------------
1   John  {"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]} 

-- Returns value too with the string represented as JSON

select id, name, interests from user 
where interests->'likes'->'[0]' = json_quote('skating');

id  name  interests                                                   
--  ----  ------------------------------------------------------------
1   John  {"likes": ["skating", "reading", "swimming"], "dislikes": ["cooking"]}
```

We can also the operators to get specific fields like queries.

```sql
-- List of first and last preference of users with skating as their first preference

select id, name, 
       interests->'likes'->>'[0]' as first_preference, 
       interests->'likes'->>'$[#-1]' as last_preference 
       from user where interests->'likes'->>'[0]' = 'skating';

id  name  first_preference  last_preference
--  ----  ----------------  ---------------
1   John  skating           swimming
```

We can also use the operators in sorting and grouping

```sql
-- List of users sorted by their first preference

select id, name, 
       interests->'likes'->>'[0]' as first_preference
       interests->'likes'->>'$[#-1]' as last_preference 
       from user order by interests->'likes'->>'[0]';

id  name  first_preference  last_preference
--  ----  ----------------  ---------------
2   Kate  reading           swimming     
3   Jim   reading           swimming     
1   John  skating           swimming

--  List of users grouped by their first preference

select id, name, 
       interests->'likes'->>'[0]' as first_preference, 
       interests->'likes'->>'$[#-1]' as last_preference 
       from user group by interests->'likes'->>'[0]';

id  name  first_preference  last_preference
--  ----  ----------------  ---------------
2   Kate  reading           swimming     
1   John  skating           swimming
```

### Indexing

Index can also be created for a given expression thus making the query efficient. Once we create an index for querying by first preference the index is used for 

```sql
-- Query plan without index

explain query plan select id, name, interests from user 
where interests->'likes'->>'[0]' = 'skating';
QUERY PLAN
--SCAN user


-- Create index on first preference of a user

create index idx_first_preference on user(interests->'likes'->>'[0]');

explain query plan select id, name, interests from user 
where interests->'likes'->>'[0]' = 'skating';
QUERY PLAN
--SEARCH user USING INDEX idx_first_preference (<expr>=?)

explain query plan select id, name, interests from user 
where interests->'likes'->>'[1]' = 'skating';
QUERY PLAN
--SCAN user
```

### Conclusion

Enabling JSON by default and the new operators in 3.38.0 improve adoption and ergonomics of using JSON in SQLite. PostgreSQL has more rich query support which will be hopefully added in future releases.