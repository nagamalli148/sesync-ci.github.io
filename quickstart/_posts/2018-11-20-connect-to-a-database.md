---
# Created by Rachael Blake and Ian Carroll on 2018-11-19
title: Connect to a Database Server
tags:
  - Database
  - R
  - Python
  - Shell
  - SSH
---

This Quick Start guide will walk you through establishing a connection
to a database on SESYNC's server. Access to your pursuit's relational
database management system (RDBMS) requires communication between a
server application (PostgreSQL or MySQL) and a client application
(RStudio, Jupyter, psql, etc.).

Each database maintained by SESYNC resides on a given host identified with
 `<HOSTNAME>.research.sesync.org`. Each database on a server has a unique name,
and only specific users known to the server are allowed password
protected access. The instructions below are specific to a PostgreSQL
database, but analagous instructions apply to a MySQL database.

When the Cyberinfrastructure staff creates a database, we will share a
configuration file with the host's name, user name, and password. This
keeps the information you don’t want to share private, and means you
don’t have to constantly enter host URLs, user names, and
passwords. If you ever move your database, there's also only one place
you need to update the configuration.

Configuration will be stored in the file
`/nfs/<PROJECT>-data/.pg_service.conf` as:

```bash
[<SERVICE>]
host=<HOSTNAME>.research.sesync.org
dbname=<DBNAME>
user=<PROJECT>
password=<PASSWORD>
```

NOTE: By default, `<SERVICE>`, `<DBNAME>`, and `<PROJECT>` will all be the same. Spaces are not allowed in this file.

The database client reads this file to verify your
credentials. Instructions for establishing a connection via RStudio,
Jupyter, or the command line are below.

### Access from [RStudio Server](https://rstudio.sesync.org)

The [RPostgres](https://CRAN.R-project.org/package=RPostgres) package is the
[DBI](https://CRAN.R-project.org/package=DBI) compliant library you need.
  
```r
library("RPostgres")
```
  
The connection to the "service" (the value between square brackets in the
configuration file) can be established using:
  
```r
mydb <-  dbConnect(Postgres(), service = "<SERVICE>")
```
  
The password will be retrieved from the configuration file if RStudio knows where
to look. The first place checked is in your home directory (i.e. it looks for the
file "\~/.pg_service.conf"). After that, it looks for the file in the location 
specified by the [PGSERVICEFILE](https://www.postgresql.org/docs/9.0/libpq-pgservice.html)
environment variable. For a version controlled RStudio project, the best
practice is to add the line `PGSERVICEFILE=/nfs/<PROJECT>-data/.pg_service.conf` to
a plain text file named ".Renviron" in the project's root directory. After doing that,
close and reopen your project to read in the environment variable.

If you're looking at a database for the first time, you may want to list all the 
tables in the database:

```r
dbListTables(mydb)
```  
  
Now a query can be created to retrieve data from, for example, the `neighborhoods`
table in your database:

```r
qry <- dbGetQuery(mydb, "SELECT * FROM neighborhoods LIMIT 10")
```

NOTE: To rerun `dbGetQuery` and reassign the result, you must first
clear the existing result using the following code: `dbClearResult(q)`

When finished, disconnect from the database.

```r
dbDisconnect(mydb)
```

### Access from [Jupyter Server](jupyter.sesync.org)

The [psycopg2](https://pypi.org/project/psycopg2/) package is a
popular wrapper for the interface to PostgreSQL databases. The
[pandas](https://pypi.org/project/pandas/) package is used for
manipulating tabular data in Python.

```python
import psycopg2
import pandas as pd
``` 
  
The connection can be established using:
  
```python
mydb = psycopg2.connect("service=<SERVICE>")
```
  
NOTE: The password will be retrieved from the configuration file at the path specified
by the [PGSERVICEFILE](https://www.postgresql.org/docs/9.0/libpq-pgservice.html)
environment variable. You can set this variable to the correct path in your Jupyter notebook by running
a cell that contains:

```python
%env PGSERVICEFILE=/nfs/<PROJECT>-data/.pg_service.conf
```

Pandas will read the results of an SQL query of the database into a
`DataFrame`:

```python
df = pd.read_sql('SELECT * FROM neighborhoods LIMIT 10', con=mydb)
```

If you're diving into a new database for the first time, you may want to
list all available tables:
  
```python
cur = mydb.cursor()
cur.execute("""
  SELECT table_name FROM information_schema.tables
  WHERE table_schema = 'public'""")
for table in cur.fetchall():
  print(table)
```
  
Now a query can be created to retrieve data from the `neighborhoods`
table in the database on urban green spaces:  

```python
df = pd.read_sql('SELECT * FROM neighborhoods LIMIT 10', con=mydb)
```
  
Pandas reads the results of an SQL query of the database into a
`DataFrame`.
  
When finished, disconnect from the database.
    
```python
mydb.close()
```
   
### Access from local RStudio or Python via SSH Tunneling
  
If you would like to use your local installation of RStudio or Python
on your own computer, you need to establish a secure connection via
SSH Tunneling. The purpose of the "tunnel" is to make a server you can
connect to (ssh.sesync.org) route connection requests to the database
server, which you cannot reach directly from your local RStudio or
Python installation.

You will need the host name from your ".pg_service.conf" file, but `<USERNAME>` below refers
to your SESYNC username. Open a command prompt or terminal on 
your machine and establish the tunnel with:
  
```bash
ssh -L 5433:<HOSTNAME>.research.sesync.org:5432 <USERNAME>@ssh.sesync.org
```

Enter your SESYNC password when prompted. If successful, your terminal
will display something like:
  
```bash
$ ssh -L 5433:<HOSTNAME>.research.sesync.org:5432 <USERNAME>@ssh.sesync.org
Ubuntu 16.04.5 LTS
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
##############################################################################
###                                                                        ###
###   SESYNC SSH Gateway                                                   ###
###   * Do not use this machine for processing                             ###
###   * Store all data and source code in your project space /nfs/PROJECT  ###
###   * See https://cyberhelp.sesync.org for more information              ###
###                                                                        ###
##############################################################################
Last login: Fri Nov 16 14:52:20 2018 from 192.168.192.177
<USERNAME>@sshgw02:~$ 
```
  
From this machine, you can use the command line client to confirm
you have access to the database:
  
```bash
PGSERVICEFILE=/nfs/<PROJECT>-data/.pg_service.conf psql service=<SERVICE>
```

More on `psql` below, what you really want though is to use your
secure connection to the SESYNC database server from the RStudio or
Python instalation on your own computer. The only thing missing is
the connection information, so copy the
".pg_service.conf" file to your own computer and edit the
server information to point at the tunnel:

```bash
[<SERVICE>]
host=localhost
port=5433
dbname=<DBNAME>
user=<PROJECT>
password=<PASSWORD>
```

On macOS, save this file in your home directory. On Windows, create
the folder `%APPDATA%\postgresql`, and save the configuration file
there. Establishing a connection from your local installation of
RStudio or Python should now work as above. NOTE: Reading data from
the database server will be slow if you are on a slow internet
connection.

### Access from the Command Line

The `psql` command line utility is available for your use on
ssh.sesync.org. From the Terminal (macOS) or Bash Shell (Windows) 
on your computer, the SESYNC user `<USERNAME>` would access the 
server via SSH with:

```r
ssh <USERNAME>@ssh.sesync.org
```

Then type in your password when prompted--you won't see any typing
going on so type carefully! You should now be connected to the gateway
server via SSH.

Now you need to log in to the PostgreSQL database using the host name, database
name, and database password. These are all saved in a ".pg_service.conf" file created
by SESYNC staff. You need to set the [environment
variable](https://www.postgresql.org/docs/9.0/libpq-pgservice.html).
  
Then the connection can be established by launching `psql` and 
specifying the service by typing this command: 
  
```bash
PGSERVICEFILE=/nfs/<PROJECT>-data/.pg_service.conf psql service=<SERVICE>
```
  
You should now be connected to your database.  Your shell will look something like:  
  
```bash
<USERNAME>@sshgw02:~$ psql service=<SERVICE>
psql (9.5.14, server 9.3.24)
SSL connection (protocol: TLSv1.2, cipher: DHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

<DBNAME>=> 
```
  
The `psql` utility allows "backslash commands" and SQL statements.  For example
if you typed
  
```bash
\d
```
a description of the database is printed.  Your shell will look something like:    
  
```bash
<DBNAME>=> \d
               List of relations
 Schema |       Name        | Type  |   Owner
--------+-------------------+-------+-----------
 public | geography_columns | view  | postgres
 public | geometry_columns  | view  | postgres
 public | neighborhoods     | table | jdoe
 public | raster_columns    | view  | postgres
 public | raster_overviews  | view  | postgres
 public | spatial_system    | table | postgres
(6 rows)

<DBNAME>=>
```

Again supposing a table `neighborhoods` exists, you could directly
enter the SQL statement or query to print all columns of its first 10
records.

```bash
SELECT * FROM neighborhoods LIMIT 10;
```
  
Your shell will look something like:  
  
```bash
<DBNAME>=> SELECT * FROM neighborhoods LIMIT 10;
 id |            name             | city_id | admin1_id | country_id | region_id
----+-----------------------------+---------+-----------+------------+-----------
  1 | Western Addition            |         |           |            | WAD
  2 | East Seaside                |         |           |            | EAS
  3 | Portside                    |         |           |            | PRT
  4 | Edwardsville                |         |           |            | EWV
  5 | Lancaster                   |         |           |            | LCR
  6 | Middle Marlborough          |         |           |            | MMB
  7 | North Ascot                 |         |           |            | NAS
  8 | Percyville                  |         |           |            | PCV
  9 | Smallerton                  |         |           |            | STN
 10 | South Amherst               |         |           |            | SAM
(10 rows)

<DBNAME>=>
```
  
When finished, disconnect from the database by typing:

```bash
 \q
```

To close the SSH connection when you are done, type `exit` and then press
ENTER.  To close the shell, simply close the Terminal or Bash Shell
application.
