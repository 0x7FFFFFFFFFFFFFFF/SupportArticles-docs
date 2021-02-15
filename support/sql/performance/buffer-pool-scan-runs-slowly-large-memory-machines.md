---
title: Buffer pool scan runs slowly on large memory machines
description: This article describes that scan the SQL Server buffer pool may take a long time on large memory machines.
ms.date: 01/15/2021
ms.prod-support-area-path: Performance
ms.reviewer: daleche
ms.topic: article
ms.prod: sql 
---
# Operations that scan SQL Server buffer pool are slow on large memory machines

This article describes that scan the SQL Server buffer pool may take a long time on large memory machines.

_Applies to:_ &nbsp; SQL Server  
_Original KB number:_ &nbsp; 4566579

## Summary

Certain operations in SQL Server trigger a scan of the buffer pool (the cache that stores database pages in memory). Here are some operations that may trigger a buffer pool scan:

- Database startup
- Database shutdown/restart
- AG failover
- Dropping a database
- Removing a file from a database
- Full/Differential Backup of a database
- Restoring a database
- Transaction log restore
- Online restore
- DBCC CheckDB/CheckTable

On systems with a large amount of memory (1 TB or higher), scanning the buffer pool takes a long time, which slows down the operation that triggered the scan.

There's currently no fix for this issue. If it is critical that the operation in question complete quickly, consider clearing the buffer pool using these commands: `USE <DatabaseName>; CHECKPOINT; GO`.

If the server has more than one database, repeat the commands above for all user databases on the server.

Once all the databases on the server have been checkpointed, run the command `DBCC DROPCLEANBUFFERS`;

> [!WARNING]
> Dropping clean buffers from the buffer pool removes all non-modified database pages from memory. This requires subsequent queries to read the data from the database files on disk and could cause severe performance degradation.

## More information

For more information on issues that may arise from large buffer pools, see [SQL Server : large RAM and DB Checkpointing](https://techcommunity.microsoft.com/t5/sql-server-support/sql-server-large-ram-and-db-checkpointing/ba-p/318973).
