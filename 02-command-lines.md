# Sqoop Command Line Part II

Table of Contents
===================

* [Scheduling Sqoop Jobs with Oozie](#scheduling-sqoop-jobs-with-oozie)
* [Importing Data Directly into Hive](#importing-data-directly-into-hive)
* [Using Partitioned Hive Tables](#using-partitioned-hive-tables)
* [Replacing Special Delimiters During Hive Import](#replacing-special-delimiters-during-hive-import)
* [Using the Correct NULL String in Hive](#using-the-correct-null-string-in-hive)
* [Importing Data into HBase, Hadoop's real-time database](#importing-data-into-hbase-hadoops-real-time-database)
* [Importing All Rows into HBase](#Importing-All-Rows-into-HBase)
* [Improving Performance When Importing into HBase](#improving-performance-when-importing-into-hbase)


# Scheduling Sqoop Jobs with Oozie

'''
Problem
You are using Oozie in your environment to schedule Hadoop jobs and would like to
call Sqoop from within your existing workflows.
'''

'''
Solution
Oozie includes special Sqoop actions that you can use to call Sqoop in your workflow.
For example:
'''

<workflow-app name="sqoop-workflow" xmlns="uri:oozie:workflow:0.1">
	...
	<action name="sqoop-action">
		<sqoop xmlns="uri:oozie:sqoop-action:0.2">
			<job-tracker>foo:8021</job-tracker>
			<name-node>bar:8020</name-node>
			<command>import --table cities --connect ...</command>
		</sqoop>
		<ok to="next"/>
		<error to="error"/>
	</action>
	...
</workflow-app>


# Importing Data Directly into Hive

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hive-import

# Using Partitioned Hive Tables

'''
Problem
You want to import data into Hive on a regular basis (for example, daily), and for that
purpose your Hive table is partitioned. You would like Sqoop to automatically import
data into the partition rather than only to the table.
'''

'''
Solution
Sqoop supports Hive partitioning out of the box. In order to take advantage of this
functionality, you need to specify two additional parameters: --hive-partition-key,	
which contains the name of the partition column, and --hive-partition-value, which
specifies the desired value. For example, if your partition column is called day and you
want to import your data into the value 2013-05-22, you would use the following
command:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hive-import \
	--hive-partition-key day \
	--hive-partition-value "2013-05-22"

# Replacing Special Delimiters During Hive Impor

'''
Problem
You’ve imported the data directly into Hive using Sqoop’s --hive-import feature. When
you call SELECT count(*) FROM your_table query to see how many rows are in the
imported table, you get a larger number than is stored in the source table on the relational
database side.
'''

'''
Solution
This issue is quite often seen when the data contains characters that are used as Hive’s
delimiters. You can instruct Sqoop to automatically clean your data using --hive-dropimport-
delims, which will remove all \n, \t, and \01 characters from all string-based
columns:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hive-import \
	--hive-drop-import-delims

'''
If removing the special characters is not an option in your use case, you can take advantage
of the parameter --hive-delims-replacement, which will accept a replacement
string. Instead of removing separators completely, they will be replaced with a
specified string. The following example will replace all \n, \t, and \01 characters with
the string SPECIAL:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hive-import \
	--hive-delims-replacement "SPECIAL"	


# Using the Correct NULL String in Hive

'''
Problem
You’ve imported your data into Hive using Sqoop and now you’re trying to query it.
However, you can see that some columns contain the correct NULL value but some contain
the string literal null and are not selected using the expression column IS NULL.
'''

'''
Solution
Due to differences in the default NULL substitution string between Sqoop and Hive, you
have to override the Sqoop default substitution strings to be compatible with Hive. For
example:
'''

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hive-import \
	--null-string '\\N' \
	--null-non-string '\\N'

# Importing Data into HBase, Hadoop’s real-time database.

sqoop import \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hbase-table cities \
	--column-family world	

# Importing All Rows into HBase

sqoop import \
	-Dsqoop.hbase.add.row.key=true \
	--connect jdbc:mysql://mysql.example.com/sqoop \
	--username sqoop \
	--password sqoop \
	--table cities \
	--hbase-table cities \
	--column-family world

# Improving Performance When Importing into HBase

'''
Problem
Imports into HBase take significantly more time than importing as text files in HDFS.
'''

'''
Solution
Create your HBase table prior to running Sqoop import, and instruct HBase to create
more regions with the parameter NUMREGIONS. For example, you can create the HBase
table cities with the column family world and 20 regions using the following
command:
'''

hbase> create 'cities', 'world', {NUMREGIONS => 20, SPLITALGO => 'HexString
Split'}	