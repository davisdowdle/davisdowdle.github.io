---
layout: post
title:  Make a MySQL Connection in Python!
author: Davis Dowdle
description: Here's how you can create a MySQL connection and pull queries in Python.
image: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCfkWw2A2cc_wd__da3HxhRZOYTUBevkf0OA&usqp=CAU"
--- 

#### Note: MySQL should usually be installed locally prior to the below procedures. Because this particular MySQL server is public (and a local version of MySQL may not be required), a tutorial for this will not be covered below. 

Python on its own is an extremely powerful resource. What makes Python versatile and useful though is its ability to pair with other programs, including database infrastructures. As such, in order to use Python to the fullest--and make a good impression of yourself--you'll need to know how to exploit this basic aspect of the world's most popular programming language. 

One of the most common Python connections is to MySQL, a ubiquotous database management system operated through SQL commands. Here's how to establish a MySQL connection and draw query output into a pandas dataframe in 3 easy steps.

## Step 1: Install/Import necessary packages.

For this process, you'll need `mysql.connector` and `pandas`. After installation, import these two packages.

![Import]({{site.url}}.{{site.baseurl}}/assets/images/blogpic1.png)

If you run into a problem with this first step, it's likely you haven't installed the packages yet. Just once, run `pip install <package name>` for the appropriate packages you need to install. After that, the code should run smoothly and you should be ready to start your connection.

## Step 2: Establish the MySQL connection.

In order to successfully establish the connection, you'll need at least the host and user associated with the MySQL database. Many require a password and some a port number. Ideally, the database will also be known to jump right into querying.

Use the `connect` method of the `mysql.connector` object and supply the necessary arguments (host, user, password, port, database, etc.) to make the connection. Assigning the connection to an object is the most efficient.

In this example, the MySQL database known as 'Rfam' is publicly available. Anyone can access it, so there is no password. You should be able to connect yourself with the code below if you've followed the above instructions correctly.

![Connect]({{site.url}}/{{site.baseurl}}/assets/images/blogpic2.png)

## Step 3: Define and extract the SQL query.

Now, write the SQL code you would like to pull from the database. Below is a sample query drawn from the database in the connection established above. If you'd like to get more acquainted with this database and write your own query or copy the query below, more details can be found at the following address: <a href="https://docs.rfam.org/en/latest/database.html" target="_blank">Rfam Database</a>

After defining the query, the simple `pd.read_sql()` command with the query as the first argument and the connection as the second argument will draw the output from the database.

![Extract]({{site.url}}/{{site.baseurl}}/assets/images/blogpic3.png)

Ta-da! If you used the query above, your output should look similar to below. 

![Output]({{site.url}}/{{site.baseurl}}/assets/images/blogpic4.png)

Now that you've got the query, you can save it as an object and modify it as a regular pandas dataframe. 

## Next Steps:

Now you can see that the code to draw MySQL queries is not complicated. Hopefully you tried the example above on your own and saw the connector in action. If you didn't, be sure to give it a try. 

Almost always, a connection is going to be more efficient than generating intermediate files from SQL queries, managing those, and then reading them into Python. For example, if your employer asks you to analyze some MySQL data in Python, you'd do much better to get the server details and extract it directly from the database through a connection instead of querying the data in MySQL, sending it to a CSV, and then reading it into Python. Trust me, it will save you time and storage in the long run and you'll look good doing it. ;)

However, it can be complicated to figure out some of the arguments of a connection. Try building your own MySQL stucture and connecting to it so you can experiment and figure out potential challenges to making MySQL connections. Make sure you have fun while you do it too!