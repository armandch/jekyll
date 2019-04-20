---
layout: post
title:  Oracle unable to extend temp segment in tablespace
author: I-Fan Chen
date:   2019-04-19 01:14:50 -0400
categories: Technology
---

In order to test a data conversion tool which my team had been developing for weeks, we wanted to import the latest database dump from production to our develop environment so we can run the tool against the latest data and assess the conversion result.

The database in our environment was Oracle 11g Release 2, and the DMP file to be imported was about 34 GB.  I let the import process ran overnight, only to realize next morning that it failed.  I checked the alert log which in our case is under `/ora01/app/oracle/diag/rdbms/orcl/orcldv/alert/log.xml` and realized there was an error in the log: `ORA-1652: unable to extend temp segment by 1024 in tablespace`.  At first this seemed weird to me but after talking to our DBA I realized this actually meant the datafile of the tablespace I was using, instead of system temp, reached its size limit.

The reasonable solution to this problem was to add more datafile to my tablespace.  However I wanted to make sure the datafile really reached its limit.  So I ran the following query to find out the block size of my database:

```sql
select value from v$parameter where name = 'db_block_size';
```

The result is 8,192.  According to a table I stole from StackOverflow, my maximum datafile size was 32 GB:

```
Block Sz   Max Datafile Sz (Gb)   Max DB Sz (Tb)

--------   --------------------   --------------

   2,048                  8,192          524,264

   4,096                 16,384        1,048,528

   8,192                 32,768        2,097,056

  16,384                 65,536        4,194,112

  32,768                131,072        8,388,224
```

And yes, the size of my datafile had indeed reached its limit.  So I proceeded to add one more datafile to my tablespace by running the following query:

```sql
alter tablespace my_tablespace
  add datafile 'mytablespace02.dbf' size 10m autoextend on maxsize unlimited;
```

And I can confirm the datafile was added by running the query below:

```sql
select bytes/1024/1024 as mb_size,
       maxbytes/1024/1024 as maxsize_set,
       x.*
from   dba_data_files x;
```

After resume importing the dump file, everything worked like a charm.
