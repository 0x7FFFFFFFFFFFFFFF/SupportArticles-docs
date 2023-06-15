---
title: SQL Transaction log grows due to long running transactions
description: This article helps you monitor the transaction log growth caused by long running transactions, and terminate those transactions if necessary, for a CDC enabled database.
ms.date: 06/15/2023
ms.custom: sap:Change data capture
ms.reviewer: abhtiwar
ms.prod: sql
---
# SQL Transaction log grows due to long running transactions when you use Change data capture

This article helps you monitor and resolve the problem where you notice continuous transaction log growth due to long running transactions, for a CDC enabled database.

## Symptoms

Consider the following scenario:

- You enable Change data capture (CDC) feature on a database.
- The source of change data for Change data capture is transaction log. As inserts, updates, and deletes are applied to tracked source tables, entries that describe those changes are added to the log.
- The transaction log, on the database grows due to long running transactions.

In this scenario, the database transaction log file grows gradually, leading to excessive transaction log space consumption. Once the transaction log size reaches the max defined limit, writes to the database fails.

## Cause

On a CDC enabled database, capture job latency holds up the log truncation to ensure changes can be captured from transaction log to the CDC change tables, preventing any loss of change data.

## Workaround

To work around this issue, run the below query and specify the transaction log threshold and time interval for monitoring the transaction log. If necessary, terminate transactions by setting `@kill_oldest_tran = 1`.

###### Transact-SQL script

```sql
-- Log Transactions that generated Txlog over this size
declare @transaction_log_bytes_used int = 5242880 -- 5MB (UPDATE)
-- Log full threshold
declare @log_full_threshold int = 30        -- Percent (UPDATE)
-- Kill Oldest Tran (0 = FALSE or 1 = TRUE)
declare @kill_oldest_tran bit = 0;          --(UPDATE)
-- Log Transactions over this duration
declare @active_tran_time_minutes int = 15  --(UPDATE)
-- this specifies the loop delay, format is Hours:minutes:seconds
declare  @delay varchar(9) = '00:10:00' 
declare @runtime datetime
declare @starttime datetime
declare @msg nvarchar(100)
declare  @oldest_tran_id bigint
		, @oldest_tran_session_id int
		, @oldest_tran_begin_time datetime
		, @killstr nvarchar(100)


IF OBJECT_ID('tblDiagLongTransactions') is NULL
BEGIN
	CREATE TABLE tblDiagLongTransactions 
	(
		[datacollectiontime] [datetime] NOT NULL ,
		[transaction_id] [bigint] NOT NULL,
		[name] [nvarchar](32) NOT NULL,
		[transaction_begin_time] [datetime] NOT NULL,
		[transaction_type] [int] NOT NULL,
		[transaction_state] [int] NOT NULL,
		[session_id] [int] NOT NULL,
		[is_user_transaction] [bit] NOT NULL,
		[database_transaction_log_bytes_used] [bigint] NOT NULL,
		[login_time] [datetime] NOT NULL,
		[last_request_start_time] [datetime] NOT NULL,
		[last_request_end_time] [datetime] NULL,
		[transaction_isolation_level] [smallint] NOT NULL,
		[host_name] [nvarchar](128) NULL,
		[nt_user_name] [nvarchar](128) NULL,
		[command] [nvarchar](32) NULL,
		[status] [nvarchar](30) NULL,
		[cpu_time] [int] NULL,
		[total_elapsed_time] [int] NULL,
		[Transaction_time_in_mins] [int] NULL,
		[logical_reads] [bigint] NULL,
		[wait_time] [int] NULL,
		[wait_type] [nvarchar](60) NULL,
		[wait_resource] [nvarchar](256) NULL,
		[blocking_session_id] [smallint] NULL,
		[program_name] [nvarchar](128) NULL,
		[granted_query_memory] [int] NULL,
		[writes] [bigint] NULL,
		[Request Reads] [bigint] NULL,
		[Session Reads] [bigint] NOT NULL,
		[Session Logical Reads] [bigint] NOT NULL,
		[statement_text] [nvarchar](max) NULL,
		[batch_text] [nvarchar](max) NULL,
		[objectid] [int] NULL,
		[query_hash] binary(8),
		[query_plan_hash] binary(8),
		[mostrecentsqltext] [nvarchar](max) NULL
	) ON [PRIMARY]
END


while (1=1)
begin
	-- Check if the database log used space is over the threshold

		SET @runtime = GETDATE()
		insert into tblDiagLongTransactions
		select distinct @runtime AS datacollectiontime,
		atr.transaction_id
		,atr.name
		,transaction_begin_time
		,transaction_type
		,transaction_state
		,dsr.session_id
		,dsr.is_user_transaction
		,dtr.database_transaction_log_bytes_used
		,s.login_time
		,s.last_request_start_time
		,s.last_request_end_time
		,s.transaction_isolation_level
		, s.host_name
		, s.nt_user_name
		, r.command
		, r.status
		, r.cpu_time
		, r.total_elapsed_time
		, DATEDIFF(mi,transaction_begin_time, getdate()) as 'Transaction_time_in_mins'
		, r.logical_reads
		, r.wait_time
		, r.wait_type
		, r.wait_resource
		, r.blocking_session_id
		, s.program_name
		, r.granted_query_memory
		, r.writes
		, r.reads [Request Reads]
		, s.reads [Session Reads]
		, s.logical_reads [Session Logical Reads]
		, (REPLACE(REPLACE(REPLACE(REPLACE(SUBSTRING(qt.text, r.statement_start_offset / 2 + 1,
												(CASE WHEN r.statement_end_offset = -1
														THEN LEN(CONVERT(NVARCHAR(MAX), qt.text)) * 2
														ELSE r.statement_end_offset
												END - r.statement_start_offset) / 2), ' ', ''), CHAR(13),
								''), CHAR(10), ''), CHAR(9), '')) AS statement_text
		, SUBSTRING(REPLACE(REPLACE(REPLACE(REPLACE(qt.text, ' ', ''), CHAR(13), ''), CHAR(10), ''), CHAR(9),
						''), 1, 256) AS batch_text
		, qt.objectid
		,query_hash
		,query_plan_hash
		,mqt.text
		from sys.dm_tran_active_transactions atr
		inner join sys.dm_tran_database_transactions dtr on atr.transaction_id = dtr.transaction_id
		inner join sys.dm_tran_session_transactions dsr on atr.transaction_id = dsr.transaction_id
		left outer join sys.dm_exec_sessions s on dsr.session_id = s.session_id
		left outer join sys.dm_exec_connections conn on s.session_id = dsr.session_id
		LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
		OUTER APPLY sys.dm_exec_sql_text(r.sql_handle) AS qt
		OUTER APPLY  sys.dm_exec_sql_text(conn.most_recent_sql_handle) as mqt
		where 	s.session_id != @@spid
		and atr.transaction_type != 2
		and (database_transaction_log_bytes_used > @transaction_log_bytes_used
		or datediff(minute,transaction_begin_time,getdate()) > @active_tran_time_minutes)
	
	-- Check Log full threshold
	if @kill_oldest_tran=1
	BEGIN
		IF exists(select 1 from sys.dm_db_log_space_usage
				  where used_log_space_in_percent >=@log_full_threshold)
		BEGIN

			select top 1
			 @oldest_tran_id=atr.transaction_id
			,@oldest_tran_begin_time=transaction_begin_time
			,@oldest_tran_session_id=dsr.session_id
			from sys.dm_tran_active_transactions atr
			left outer join sys.dm_tran_database_transactions dtr on atr.transaction_id = dtr.transaction_id
			left outer join sys.dm_tran_session_transactions dsr on atr.transaction_id = dsr.transaction_id
			left outer join sys.dm_exec_sessions s on dsr.session_id = s.session_id
			where 	dsr.session_id != @@spid
			and is_user_transaction=1
			order by transaction_begin_time desc

			select @oldest_tran_id as TranID,@oldest_tran_begin_time as TranbeginTime,@oldest_tran_session_id as SessionID
			set @killstr = 'KILL '+ cast(@oldest_tran_session_id as varchar(100))
			PRINT @killstr
			
            -- Kill oldest tran
			EXEC (@killstr)
			-- Checkpoint
			CHECKPOINT
		END
	END

	-- Change the polling interval as required
	waitfor delay @delay
 
end
```
