Partitioning
============

Partitioning is used to put table A and table B on different servers so as to balance the load on the DB. On a Web system, it is often the case that the database is the bottleneck.

As the amount of data in the table rests in the memory of the DB server, equivalent to disk I/O, increases it creates a loss of performance. The simplest answer may be to expand the memory (because memory is cheap these days). However, there are various problems, so it may not be so simple.

It may be possible to increase the DB server itself, and it is worth considering partitioning of the server. However, this is a difficult issue, with the scale depending on the size of the DB, so I will not discuss it here.  Many helpful books have been published on the subject.

The TreeFrog Framework provides a mechanism for partitioning, giving easy access data on a different DB server.

Partitioning by SqlObject
-------------------------

As a prerequisite, we use table A for host A, and table B for host B.

First, make separate database configuration files describing the host connection information. The file names must be databaseA.ini and databaseB.ini respectively. They go in the config directory. For the contents of the file, write the appropriate values. Here is an example of the way the dev section might be defined.

.. code-block:: ini
  
  [dev]
  DriverType=QMYSQL
  DatabaseName=foodb
  HostName=192.168.xxx.xxx
  Port=3306
  UserName=root
  Password=xxxx
  ConnectOptions=

Next define the file names of the database configuration files in the application configuration file (*application.ini*). Use DatabaseSettingsFiles, with the values written side by side, separated by spaces.

.. code-block:: ini
  
  # Specify setting files for databases.
  DatabaseSettingsFiles=databaseA.ini  databaseB.ini

The database IDs follow the sequence 0, 1, 2, … as per. This example,

  + The database ID of the host A : 0
  + The database ID of the host B : 1

Then, set the header file of SqlObject with this database ID.
Edit the header file that is generated by the generator, you can override the databaseId() method to return the database ID.

.. code-block:: c++

  class T_MODEL_EXPORT BlogObject : public TSqlObject, public QSharedData
  {
      
      int databaseId() const { return 1; }  // returns ID 1
  
  };

By doing this, queries for BlogObject will be issued to host B. After that, we can just use the SqlObject as in the past.

The example here was made with two DB servers, but is also compatible with three or more DB servers, simply by adding the DatabaseSettingsFiles in the configuration file.

Thus, partitioning can be applied without changing the logic of the model and controller.

Querying Partitioned Tables
---------------------------

As discussed in the :ref:`SQL query <model_sql_query>`, is not use the full TSqlQuery object is not just used to issue a query on its own, but you can use it to apply to the partitioning.

As follows, the database ID is specified in the second argument to the constructor.

.. code-block:: c++
  
  TSqlQuery query("SELECT * FROM foo WHERE ...",  1);  // specifies database ID 1
  query.exec();

This query is then therefore executed to host B.
