---
layout: post
title:  Python MySQL Connection
author: Davis Dowdle
description: Here's how you can create a MySQL connection and pull queries in Python.
image: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCfkWw2A2cc_wd__da3HxhRZOYTUBevkf0OA&usqp=CAU"
--- 

#### Note: MySQL should be installed locally prior to attempting this. This will not be covered below. Before moving on, make sure your local computer has MySQL prepared.

Python on its own is an extremely powerful resource. What makes Python versatile and useful though is its ability to pair with other programs, including database infrastructures. As such, in order to use Python to the fullest--and make a good impression of yourself--you'll need to know how to exploit this basic aspect of the world's most common programming language. 

One of the most common Python connections is to MySQL, a ubiquotous database management system operated through SQL commands. Here's how to establish a MySQL connection and draw query output into a pandas dataframe. 

## Step 1: Install/Import necessary packages.

For this process, you'll need mysql.connector and pandas. After installation, the code below will import the two.

![Import]({{site.url}}.{{site.baseurl}}/assets/images/blogpic1.png)

## Step 2: Establish the MySQL connection.

In order to successfully establish the connection, you'll need at least the host and user associated with the MySQL database. Many require a password and some a port number. Ideally, the database will also be known to jump right into querying.

In this example, the MySQL database is publicly available. You should be able to connect yourself with the code below if you've followed the above instructions (including MySQL installation and configuration) correctly.

