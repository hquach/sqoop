# Sqoop Command Line  Part I

Table of Contents
===================

* [Transferring an Entire Table](README.md#Transferring-an-Entire-Table)
* [Specifying a Target Directory](README.md#Specifying-a-Target-Directory)
* [To specify the parent directory for all your Sqoop jobs](README.md#To-specify-the-parent-directory-for-all-your-Sqoop-jobs)
* [Importing Only a Subset of Data](README.md#Importing-Only-a-Subset-of-Data)
* [Protecting Your Password](README.md#Protecting-Your-Password)
* [Here’s an example of reading the password from a file](README.md#Here-an-example-of-reading-the-password-from-a-file)
* [Using a File Format Other Than CSV](README.md#Using-a-File-Format-Other-Than-CSV)
* [Compressing Imported Data](README.md#Compressing-Imported-Data)
* [Speeding Up Transfers](README.md#Speeding-Up-Transfers)
* [Overriding Type Mapping](README.md#Overriding-Type-Mapping)
* [Controlling Parallelism](README.md#Controlling-Parallelism)
* [Encoding NULL Values](README.md#Encoding-NULL-Values)
* [Importing All Your Tables](README.md#Importing-All-Your-Tables)
* [Excluding tables](README.md#Excluding-tables)
* [Importing Only New Data](README.md#Importing-Only-New-Data)
* [Incrementally Importing Mutable Data](README.md#Incrementally-Importing-Mutable-Data)
* [Importing Data from Two Tables](README.md#Importing-Data-from-Two-Tables)
* [Using Custom Boundary Queries](README.md#Using-Custom-Boundary-Queries)
* [Renaming Sqoop Job Instances	](README.md#Renaming-Sqoop-Job-Instances)
* [Importing Queries with Duplicated Columns](README.md#Importing-Queries-with-Duplicated-Columns)
* [Transferring Data from Hadoop](README.md#Transferring-Data-from-Hadoop)
* [Inserting Data in Batches](README.md#Inserting-Data-in-Batches)
* [Updating an Existing Data Set](README.md#Updating-an-Existing-Data-Set)
* [Updating or Inserting at the Same Time](README.md#Updating-or-Inserting-at-the-Same-Time)
* [Exporting into a Subset of Columns](README.md#Exporting-into-a-Subset-of-Columns)


# Transferring an Entire Table

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities

# Specifying a Target Directory

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--target-dir /etl/input/cities

# To specify the parent directory for all your Sqoop jobs, instead use the --warehousedir parameter:

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--warehouse-dir /etl/input/	

# Importing Only a Subset of Data

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--where "country = 'USA'"


# Protecting Your Password

Here’s a Sqoop execution that will read the password from standard input:

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--table cities \
	-P

# Here’s an example of reading the password from a file:

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--table cities \
	--password-file my-sqoop-password

# Using a File Format Other Than CSV (Avro)	

# Avro can be enabled by specifying the --as-avrodatafile parameter:

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--as-avrodatafile

# Compressing Imported Data	

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--table cities \
	--compress

# Speeding Up Transfers

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--table cities \
	--direct


# Overriding Type Mapping

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--table cities \
	--map-column-java id=Long

# Controlling Parallelism
'''
Problem
Sqoop by default uses four concurrent map tasks to transfer data to Hadoop. Transferring
bigger tables with more concurrent tasks should decrease the time required to
transfer all data. You want the flexibility to change the number of map tasks used on a
per-job basis.	
'''

'''
Use the parameter --num-mappers if you want Sqoop to use a different number of mappers.
For example, to suggest 10 concurrent tasks, you would use the following Sqoop
command:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--num-mappers 10

# Encoding NULL Values
sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--null-string '\\N' \
	--null-non-string '\\N'	


# Importing All Your Tables

sqoop import-all-tables \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop

# Excluding tables

sqoop import-all-tables \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--exclude-tables cities,countries		

# Importing Only New Data

'''
Problem
You have a database table with an INTEGER primary key. You are only appending new
rows, and you need to periodically sync the table’s state to Hadoop for further processing.
'''

'''
Solution
Activate Sqoop’s incremental feature by specifying the --incremental parameter. The
parameter’s value will be the type of incremental import. When your table is only getting
new rows and the existing ones are not changed, use the append mode.
Incremental import also requires two additional parameters: --check-column indicates
a column name that should be checked for newly appended data, and --last-value
contains the last value that successfully imported into Hadoop.
'''

'''
The following example will transfer only those rows whose value in column id is greater
than 1:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table visits \
	--incremental append \
	--check-column id \
	--last-value 1

# Incrementally Importing Mutable Data

'''
Problem
While you would like to use the incremental import feature, the data in your table is
also being updated, ruling out use of the append mode.
'''

'''
Solution
Use the lastmodified mode instead of the append mode. For example, use the following
command to transfer rows whose value in column last_update_date is greater than
2013-05-22 01:01:01:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table visits \
	--incremental lastmodified \
	--check-column last_update_date \
	--last-value "2013-05-22 01:01:01"

# Importing Data from Two Tables	

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--query 'SELECT normcities.id, \
			countries.country, \
			normcities.city \
			FROM normcities \
			JOIN countries USING(country_id) \
			WHERE $CONDITIONS' \
	--split-by id \
	--target-dir cities


# Using Custom Boundary Queries

'''
Problem
You found free-form query import to be very useful for your use case. Unfortunately,
prior to starting any data transfer in MapReduce, Sqoop takes a long time to retrieve
the minimum and maximum values of the column specified in the --split-by parameter
that are needed for breaking the data into multiple independent tasks.
'''

'''
Solution
You can specify any valid query to fetch minimum and maximum values of the --splitby
column using the --boundary-query parameter:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--query 'SELECT normcities.id, \
			countries.country, \
			normcities.city \
			FROM normcities \
			JOIN countries USING(country_id) \
			WHERE $CONDITIONS' \
	--split-by id \
	--target-dir cities \
	--boundary-query "select min(id), max(id) from normcities"


# Renaming Sqoop Job Instances	

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--query 'SELECT normcities.id, \
			countries.country, \
			normcities.city \
			FROM normcities \
			JOIN countries USING(country_id) \
			WHERE $CONDITIONS' \
	--split-by id \
	--target-dir cities \
	--mapreduce-job-name normcities

# Importing Queries with Duplicated Columns

--query "SELECT \
		cities.city AS first_city \
		normcities.city AS second_city \
	FROM cities \
	LEFT JOIN normcities USING(id)"	


# Transferring Data from Hadoop

'''
Problem
You have a workflow of various Hive and MapReduce jobs that are generating data on
a Hadoop cluster. You need to transfer this data to your relational database for easy
querying.
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--export-dir cities	

# Inserting Data in Batches

'''
Problem
While Sqoop’s export feature fits your needs, it’s too slow. It seems that each row is
inserted in a separate insert statement. Is there a way to batch multiple insert statements
together?
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--export-dir cities \
	--batch


'''
The second option is to use the property sqoop.export.records.per.statement to
specify the number of records that will be used in each insert statement:
'''

sqoop export \
	-Dsqoop.export.records.per.statement=10 \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--export-dir cities

Finally, you can set how many rows will be inserted per transaction with the sqoop.export.statements.per.transaction property:

sqoop export \
	-Dsqoop.export.statements.per.transaction=10 \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--export-dir cities

Exporting with All-or-Nothing Semantics

'''
Problem
You need to ensure that Sqoop will either export all data from Hadoop to your database
or export no data (i.e., the target table will remain empty).				
'''

'''
Solution
You can use a staging table to first load data to a temporary table before making changes
to the real table. The staging table name is specified via the --staging-table parameter.
In the below example, we set it to staging_cities:
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--staging-table staging_cities

# Updating an Existing Data Set	

'''
Problem
You previously exported data from Hadoop, after which you ran additional processing
that changed it. Instead of wiping out the existing data from the database, you prefer to
just update any changed rows.
'''

'''
Solution
You can take advantage of the update feature that will issue UPDATE instead of INSERT
statements. The update mode is activated by using the parameter --update-key that
contains the name of a column that can identify a changed row—usually the primary
key of a table. For example, the following command allows you to use the column id of
table cities:
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--update-key id


# Updating or Inserting at the Same Time

'''
Problem
You have data in your database from a previous export, but now you need to propagate
updates from Hadoop. Unfortunately, you can’t use the update mode, as you have a
considerable number of new rows and you need to export them as well.
'''

'''
Solution
If you need both updates and inserts in the same job, you can activate the so-called
upsert mode with the --update-mode allowinsert parameter. For example:
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--update-key id \
	--update-mode allowinsert

# Exporting into a Subset of Columns

'''
Problem
You have data in Hadoop that you need to export. Unfortunately, the corresponding
table in your database has more columns than the HDFS data.	
'''

'''
Solution
You can use the --columns parameter to specify which columns (and in what order)
are present in the Hadoop data. For example, to limit the export to columns country
and city, use the following command:
'''

sqoop export \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--columns country,city