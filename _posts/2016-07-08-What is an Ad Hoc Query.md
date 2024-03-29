---
tags: [concept,sql]
categories: [tech]
---

## What's an Ad Hoc Query? 

An Ad-Hoc Query is a query that cannot be determined prior to the moment the query is issued. It is created in order to get information when need arises and it consists of dynamically constructed SQL which is usually constructed by desktop-resident query tools. This is in contrast to any query which is predefine and performed routinely.  
  
The word Ad hoc comes from Latin which means "for the purpose". It generally refers to anything that has been designed to answer very specific problems. An Ad hoc committed for instance is created to deal with a particular undertaking and after the undertaking is finished, the committee is disbanded. In the same manner, an ad hoc query does not reside in the computer or the database manager but is dynamically created depending on the needs of the data user.  
  
In the past, for users to analyze various kinds of data, multiple sets of queries are being constructed. These queries are predefined under the management of a database or system administrator and so a barrier between the users’ needs and the canned information exists. As a result, the end user gets a bombardment of unrelated data in his query results. The IT resources also get a heavy toll since a user may have to execute several different queries at any given period.  
  
Today’s widely used active data warehouses accelerate retrieval of vital information to answer interactive queries in a mission critical application.  
  
Most users of data are in fact non technical people. Everyday, the retrieve seemingly unrelated data from different tables and database sources. Many ad hoc query tools exists so non technical users can execute very complex queries without trying to know what happens at the backend. Ad hoc query tools include features to support all types of query relationships which include one-to-many, many-to-one, and many-to-many. End users can easily construct complex queries using graphical user interface (GUI) navigation through object structures in a drag and drop manner.  
  
Ad hoc queries are used intensively in the internet. Search engines process millions of queries every single second from different data sources. Any keywords typed the internet user are dynamically generated with an ad hoc query against virtually any database back end. As the basic structure of an SQL statement consist of SELECT keyword FROM table WHERE conditions, an ad hoc query dynamically supplies the keyword, data source and the conditions without the user knowing it.  
  
Although issuing an ad hoc query against a database may be more efficient in terms of computer resources because using a predefined query may cause one to issue more than one query, using an ad hoc query may also have a heavy resource impact depending on the number of variables needed to be answered.  
  
To reduce impact on memory due to usage of ad hoc queries, the computer must have huge amount of memory, provide very fast devices to be used as temporary disk storage and the database manager must prevent very high memory usage ad hoc queries from being executed. Some database managers anticipate huge sort requirements by having exact match pre-calculated results sets.  
  
But in a general ad hoc environment, a user is discouraged from issuing an ad hoc query to produce a report based on millions of transactions from the last years. Instead, users may choose data from a given range.  
  
Because of the high potential of performance degradation when a complex ad hoc query is executed, database managers sometimes only provide copy of the live database to be regularly refreshed. The copy is sometimes referred to as data warehouse and the act of querying as data mining.  