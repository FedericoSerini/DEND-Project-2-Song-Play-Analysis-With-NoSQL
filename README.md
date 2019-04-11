# Project 2: Song Play Analysis With NoSQL

[![Project passed](https://img.shields.io/badge/project-passed-success.svg)](https://img.shields.io/badge/project-passed-success.svg)

## Summary
* [Preamble](#Preamble)
* [ETL process](#ETL-process)
* [How to run](#How-to-run)
* [Project structure](#Project-structure)
* [CQL queries](#CQL-queries)

-------------------------------------------

#### Preamble

This project is not focused on ETL process, but on data modeling <br> on Cassandra and how it differs from relational database data modeling.

Let's start with ``PRIMARY KEY``, in Cassandra primary keys works slightly different from RDBMS ones, <br> in fact in Cassandra, the ``PRIMARY KEY`` is made up of either just the ``PARTITION KEY`` <br> or with the addition of ``CLUSTERING COLUMNS``.
And you can have just one record unique, <br> if we insert data with same ``PRIMARY KEY`` then data will be overwritten with the latest record state

In ``WHERE`` conditions the ``PRIMARY KEY`` must be included and you can use your ``CLUSTERING KEY`` <br> to order your data
but they are not necessary, by default data will be ordered as ``DESC``


In Cassandra the ``denormalization`` process is a must, you cannot apply normalization like an RDBMS <br> because you cannot use JOINs. More the denormalization process is done more the query will <br> run faster, in fact, Cassandra has been optimized for fast writes not for fast reads. <br>
To reach the complete denormalization you have to follow the pattern ``1 Query - 1 Table``.<br> This leads to data duplication but it does not matter, the denormalization process <br> by nature itself produces data duplication.


Following the ``CAP theorem``, Cassandra embraces the ``AP`` guarantees. <br>
It provides only AP because of its structure, data are shared by nodes so if <br>
a node goes down another one can satisfy the client request but due <br>
the high number of nodes data could be not updated for each node that is why <br>
Cassandra offers only ``Eventual Consistency``

--------------------------------------------

#### ETL process

Although this is not an ETL focused project, we must care how data will be <br> ingested in our database, and how to read our input properly. <br> Our ETL process consist in read every file in /event_data, this data have to be <br> aggregated in a file called event_datafile_new.csv and after the aggregation, <br>
 the file has to be parsed and persisted on the database

--------------------------------------------

#### How to run
First of all, you need a Cassandra instance up and running <br>
Here you can find the [Binary packages](http://cassandra.apache.org/download/) for your preferred operating system  <br>
After downloading the package, If you do not know to move on just follow this [Documentation](http://cassandra.apache.org/doc/latest/getting_started/index.html) <br>

You have to install also [Python](https://www.python.org/downloads/) and [Jupyter Notebook](https://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/install.html) <br>

<b> Note: </b><br>
In this example we will not use any authorization mechanism

After installing your database, Python and Jupyter on your local machine <br>
open your terminal and type

`jupyter notebook`

It will start the service, after the service has been started just drag and drop the notebook

--------------------------------------------

#### Project structure
This is the project structure, if the bullet contains ``/`` <br>
means that the resource is a folder:

* <b> /event_data </b> - The directory of CSV files partitioned by date
* <b> /images </b> - Simply a folder with images that are used in Project_1B_ Project_Template notebook
* <b> Project_1B_ Project_Template.ipynb </b> - It is a notebook that illustrates the project step by step
* <b> event_datafile_new.csv </b> - The aggregated CSV composed by all event_data files

--------------------------------------------

#### CQL queries

<I> Query 1:  Give me the artist, song title and song's length in the music app history that was heard <br> during
 sessionId = 338 and itemInSession = 4 </I>
``` SQL
CREATE TABLE IF NOT EXISTS song_data_by_session (session_id INT,
        session_item_number INT,
        artist_name TEXT,
        song_title TEXT,
        song_length DOUBLE,
        PRIMARY KEY ((session_id, session_item_number)))
```
 In this case <b>session_id</b> and <b>session_item_number</b> are enough to
 make a record unique for our request.
 Our complete ``PRIMARY KEY`` is composed by <b>session_id</b>, <b>session_item_number </b>
 
``` SQL
SELECT artist_name, song_title, song_length
 FROM song_data_by_session
 WHERE session_id = 338 AND session_item_number =  4
```

<I> Query 2: Give me only the following: name of artist, song (sorted by itemInSession)  <br> and user (first and last name) for userid = 10, sessionid = 182 </I>
``` SQL
CREATE TABLE IF NOT EXISTS song_user_data_by_user_and_session_data (user_id INT,
        session_id INT,
        session_item_number INT,
        artist_name TEXT,
        song_title TEXT,
        user_first_name TEXT,
        user_last_name TEXT,
        PRIMARY KEY ((user_id, session_id), session_item_number))
```
In this case <b>user_id</b> and <b>session_id</b> are the ``COMPOUND PARTITION KEY``  
this allows us to have a unique ``PRIMARY KEY`` for our query, but for this request we have to 
order by session_item_number but not to query on that, so we have to declare <b>session_item_number</b> as ``CLUSTERING KEY``.  
Our complete PRIMARY KEY is composed by <b>user_id</b>, <b>session_id</b>, <b>session_item_number</b>
``` SQL
SELECT artist_name, song_title, user_first_name, user_last_name
 FROM song_user_data_by_user_and_session_data
 WHERE user_id = 10 AND session_id = 182
```

<I> Query 3: Give me every user name (first and last) in my music app history who listened <br> to the song 'All Hands Against His Own' </I>
``` SQL
CREATE TABLE IF NOT EXISTS user_data_by_song_title (song_title TEXT, user_id INT,
        user_first_name TEXT,
        user_last_name TEXT,
        PRIMARY KEY ((song_title), user_id))
```
In this case <b>song_title</b> is the ``PARTITION KEY`` and <b>user_id </b>
is the ``CLUSTERING KEY``, the request asks to retrieve the user name 
by song title, so we have to set <b>song_title</b> as ``PARTITION KEY``, but 
more users can listen to the same song so we may have many ``INSERT`` with the 
same key, Cassandra overwrites data with the same key so we need to add a ``CLUSTERING KEY`` 
because we need to have a unique record but not to query on that. 
Our complete ``PRIMARY KEY`` is composed by <b>song_title</b>, <b>user_id</b>
``` SQL
SELECT user_first_name, user_last_name
 FROM user_data_by_song_title
 WHERE song_title = 'All Hands Against His Own'
```

----------------------------
