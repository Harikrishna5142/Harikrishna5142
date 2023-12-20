- 👋 Hi, I’m @Harikrishna5142
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Harikrishna5142/Harikrishna5142 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.


ORACLE DATABASE ADMINSTRATION

MS SQL DBA COMMANDS

                 LOGINS
User create

use Sales_BI
go
CREATE USER [E83122] for login [E83122];
grant select on TB_RNE_Voucher  to [E83122]

grant select on offer_prorderwisedetails_FO_RTB to  [post-trade-core-fno]
go
grant insert on offer_prorderwisedetails_FO_RTB to  [post-trade-core-fno]
go
grant delete on offer_prorderwisedetails_FO_RTB to  [post-trade-core-fno]
go
grant update on offer_prorderwisedetails_FO_RTB to  [post-trade-core-fno]
LOGIN

CREATE LOGIN <login_name> WITH PASSWORD = '<enterStrongPasswordHere>';
GO


to find login exist

select * from [dbo].[User_Login_Details] where Login_name like '%Utkarsh%' 
	
select * from [dbo].[User_Login_Details] where Login_id='E83176'



To provide permissions on all databases
FOR SQL 2012 onwards
========================================================================================================================

USE [master]
GO
CREATE LOGIN [E84570] WITH PASSWORD=N'b@F0HFLaTE8S%ysvdqwe', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO

DECLARE @dbName VARCHAR(100)
DECLARE @sSql NVARCHAR(4000)
DECLARE @Login VARCHAR(50)

SET @Login = 'E84570'

DECLARE curDB CURSOR FOR SELECT Name FROM sys.databases WHERE database_id > 4 AND state_desc = 'ONLINE' AND Name NOT LIKE 'ReportServer%'
OPEN curDB
FETCH NEXT FROM curDB INTO @dbName
WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @sSql = 'USE [' + @dbName + '] 
					GO
					CREATE USER [' + @Login + '] FOR LOGIN [' + @Login + ']
					GO
					ALTER ROLE [db_datareader] ADD MEMBER [' + @Login + ']
					GO
					--ALTER ROLE [db_datawriter] ADD MEMBER [' + @Login + ']
					--GO
					--ALTER ROLE [db_ddladmin] ADD MEMBER [' + @Login + ']
					--GO
					--ALTER ROLE [db_owner] ADD MEMBER [' + @Login + ']
					--GO
					GRANT EXECUTE TO [' + @Login + ']
					--GO'

		PRINT @sSql

		FETCH NEXT FROM curDB INTO @dbName

	END

CLOSE curDB
DEALLOCATE curDB


========================================================================================================================
For SQL 2008
========================================================================================================================

USE [master]
GO
CREATE LOGIN [E83184] WITH PASSWORD=N'njksyijg %$12hy&%jl34', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO

DECLARE @dbName VARCHAR(100)
DECLARE @sSql NVARCHAR(4000)
DECLARE @Login VARCHAR(50)

SET @Login = 'E83184'

DECLARE curDB CURSOR FOR SELECT Name FROM sys.databases WHERE database_id > 4 AND state_desc = 'ONLINE' AND Name NOT LIKE 'ReportServer%'
OPEN curDB
FETCH NEXT FROM curDB INTO @dbName
WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @sSql = 'USE [' + @dbName + '] 
					GO
					CREATE USER [' + @Login + '] FOR LOGIN [' + @Login + ']
					GO
					EXEC sp_addrolemember N''db_datareader'', N''' + @Login + '''
					GO'
		PRINT @sSql

		FETCH NEXT FROM curDB INTO @dbName

	END

CLOSE curDB
DEALLOCATE curDB




TO CHECK DOMAIN OF THE SERVER


SELECT DEFAULT_DOMAIN() [DomainName]



RETURN 0

Blockings occurs if a particular query is running in single SP ID then need to add RETURN 0

                                  

                                
								
								
								TO FIND Memory UTILIZATION
								-----------------------------

/*Memory*/
EXEC dbo.sp_WhoIsActive
@get_transaction_info=0,
@output_column_list ='[session_id][start_time]
                  [cpu][used_memory][status][context_switches][wait_info][program_name]
                 [database_name][blocking_session_id][sql_text][host_name][open_tran_count]', 
@sort_order='[used_memory]DESC'

                                       CPU UTILIZATION
									   ----------------
/*CPU*/
EXEC dbo.sp_WhoIsActive
@get_transaction_info=0,
@output_column_list ='[dd hh:mm:ss.mss][session_id][start_time]
                  [cpu][status][context_switches][wait_info][program_name]
                 [database_name][blocking_session_id][sql_text][host_name][open_tran_count]', 
@sort_order='[CPU]DESC'


                                            CPU UTILIZATION
											-----------------

	 SELECT
		r.session_id
		,st.TEXT AS batch_text
		,SUBSTRING(st.TEXT, statement_start_offset / 2 + 1, (
				(
					CASE
						WHEN r.statement_end_offset = - 1
							THEN (LEN(CONVERT(NVARCHAR(max), st.TEXT)) * 2)
						ELSE r.statement_end_offset
						END
					) - r.statement_start_offset
				) / 2 + 1) AS statement_text
		,qp.query_plan AS 'XML Plan'
		,r.cpu_time, r.total_elapsed_time
		,r.logical_reads, r.writes, r.dop
	FROM sys.dm_exec_requests r
	CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) AS st
	CROSS APPLY sys.dm_exec_query_plan(r.plan_handle) AS qp
	ORDER BY cpu_time DESC
	
	
	                                            BACKUP CHECK
												-------------
				   
SELECT r.session_id,r.command,CONVERT(NUMERIC(6,2),r.percent_complete)
AS [Percent Complete],CONVERT(VARCHAR(20),DATEADD(ms,r.estimated_completion_time,GetDate()),20) AS [ETA Completion Time],
CONVERT(NUMERIC(10,2),r.total_elapsed_time/1000.0/60.0) AS [Elapsed Min],
CONVERT(NUMERIC(10,2),r.estimated_completion_time/1000.0/60.0) AS [ETA Min],
CONVERT(NUMERIC(10,2),r.estimated_completion_time/1000.0/60.0/60.0) AS [ETA Hours],
CONVERT(VARCHAR(1000),(SELECT SUBSTRING(text,r.statement_start_offset/2,
CASE WHEN r.statement_end_offset = -1 THEN 1000 ELSE (r.statement_end_offset-r.statement_start_offset)/2 END)
FROM sys.dm_exec_sql_text(sql_handle))) AS [SQL]
FROM sys.dm_exec_requests r WHERE command IN ('RESTORE DATABASE','BACKUP DATABASE')


SELECT
   session_id as SPID, command, a.text AS Query, start_time, percent_complete,
   dateadd(second,estimated_completion_time/1000, getdate()) as estimated_completion_time
FROM sys.dm_exec_requests r
   CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
WHERE r.command in ('BACKUP DATABASE','RESTORE DATABASE')



                                   To check backup history 
								   ------------------------
								   
								   
use msdb 



backup drives

Virtual backup= commvault
               =Rubrik
			   
			   
drive names=native backup	
  
  
                       DATABASE STATUS

sp_helpdb 	

                      OS version 

SELECT host_platform, host_distribution
FROM sys.dm_os_host_info;

TEST MAIL NOT RECEIVED

N


Msg 14641, Level 16, State 1, Procedure msdb.dbo.sp_send_dbmail, Line 81 [Batch Start Line 1]
Mail not queued. Database Mail is stopped. Use sysmail_start_sp to start Database Mail.

1.first we need to check management -databse mail-try with your mail id then

sysmail_start_sp  --- run it on msdb databse on server.

                                   RESTORE DATABASE
								   -----------------
								   
USE [master]
RESTORE DATABASE [HKClientConfig] 
FROM  DISK = N'S:\Backup\HKClientConfig_300523.bak' 
WITH  FILE = 1,  MOVE N'HKClientConfig' TO N'D:\SQL_DATA\HKClientConfig.mdf',  
MOVE N'HKClientConfig_log' TO N'L:\SQL_LOG\HKClientConfig_log.ldf',  NOUNLOAD,  REPLACE,  STATS = 5

GO


USE [master]
RESTORE DATABASE [DBA_ADMIN]
 FROM  DISK = N'S:\Backup\DBA_ADMIN FULL_15jul.bak'
 WITH  FILE = 1,  MOVE N'DBA_ADMIN' TO N'D:\SQL_Data\DBA_ADMIN.mdf',  
 MOVE N'DBA_ADMIN_log' TO N'L:\SQL_Log\DBA_ADMIN_log.LDF',  NOUNLOAD,  REPLACE,  STATS = 5

GO



USE [master]
RESTORE DATABASE [Integra] FROM  DISK = N'S:\Backup\Integra_full_backup_15jul.bak' WITH  FILE = 1,  MOVE N'NewMWIntegra_Data' TO N'D:\SQL_Data\Integra.mdf',  MOVE N'DB_Event_Logs' TO N'D:\SQL_Data\Integra_1.ndf',  MOVE N'CDC_Logs' TO N'D:\SQL_Data\Integra_2.ndf',  MOVE N'Staging' TO N'D:\SQL_Data\Integra_3.ndf',  MOVE N'Staging_Int' TO N'D:\SQL_Data\Integra_4.ndf',  MOVE N'DevIntegra_111_MWI_Imports' TO N'D:\SQL_Data\Integra_5.ndf',  MOVE N'NewMWIntegra_Log' TO N'L:\SQL_Log\Integra_6.ldf',  MOVE N'DevIntegra_111_MWI_FileStream_Data' TO N'D:\SQL_Data\Integra_7.DevIntegra_111_MWI_FileStream_Data',  NOUNLOAD,  REPLACE,  STATS = 5

GO


                              AUTO MAIL FAIL CHECK
							  
SELECT COUNT(1), Sent_Status FROM sysmail_allitems WITH (NOLOCK)
WHERE send_request_date >= '2023-03-07 00:00:00'
GROUP BY Sent_Status   -- curently failed is 29, if go above pls check


select * from  sysmail_allitems WITH (NOLOCK)
WHERE send_request_date >= '2023-03-07 00:00:00' and Sent_Status = 'failed'
order by mailitem_id desc


select * from sysmail_profile

Host Name : smtpbp.falconide.com
user name : angelbrokingltdfal
Password  : Abl@2016

management-sql server log-dataase mail-configuration mail and select step 2 in both steps and next




                                     BACKUP CHECK

DECLARE @xp_results TABLE
(
	job_id                UNIQUEIDENTIFIER NOT NULL,
	last_run_date         INT              NOT NULL,
	last_run_time         INT              NOT NULL,
	next_run_date         INT              NOT NULL,
	next_run_time         INT              NOT NULL,
	next_run_schedule_id  INT              NOT NULL,
	requested_to_run      INT              NOT NULL, -- BOOL
	request_source        INT              NOT NULL,
	request_source_id     sysname          COLLATE database_default NULL,
	running               INT              NOT NULL, -- BOOL
	current_step          INT              NOT NULL,
	current_retry_attempt INT              NOT NULL,
	job_state             INT              NOT NULL
)

DECLARE @job_execution_state TABLE 
(
	job_id                  UNIQUEIDENTIFIER NOT NULL,
	date_started            INT              NOT NULL,
	time_started            INT              NOT NULL,
	execution_job_status    INT              NOT NULL,
	execution_step_id       INT              NULL,
	execution_step_name     sysname          COLLATE database_default NULL,
	execution_retry_attempt INT              NOT NULL,
	next_run_date           INT              NOT NULL,
	next_run_time           INT              NOT NULL,
	next_run_schedule_id    INT              NOT NULL,
	running					INT              NOT NULL -- BOOL
)

DECLARE @filtered_jobs TABLE 
(
	job_id                   UNIQUEIDENTIFIER NOT NULL,
	date_created             DATETIME         NOT NULL,
	date_last_modified       DATETIME         NOT NULL,
	current_execution_status INT              NULL,
	current_execution_step   sysname          COLLATE database_default NULL,
	current_retry_attempt    INT              NULL,
	last_run_date            INT              NOT NULL,
	last_run_time            INT              NOT NULL,
	last_run_outcome         INT              NOT NULL,
	next_run_date            INT              NULL,
	next_run_time            INT              NULL,
	next_run_schedule_id     INT              NULL,
	type                     INT              NOT NULL,
	running					 INT              NOT NULL -- BOOL
)


INSERT INTO @xp_results 
EXECUTE master.dbo.xp_sqlagent_enum_jobs 1, 'sa'--, @job_id

-- SELECT * FROM @xp_results


INSERT INTO @job_execution_state
  SELECT xpr.job_id,
         xpr.last_run_date,
         xpr.last_run_time,
         xpr.job_state,
         sjs.step_id,
         sjs.step_name,
         xpr.current_retry_attempt,
         xpr.next_run_date,
         xpr.next_run_time,
         xpr.next_run_schedule_id,
		 xpr.running
  FROM @xp_results                          xpr
       LEFT OUTER JOIN msdb.dbo.sysjobsteps sjs ON ((xpr.job_id = sjs.job_id) AND (xpr.current_step = sjs.step_id)),
       msdb.dbo.sysjobs_view                sjv
  WHERE (sjv.job_id = xpr.job_id)


 -- SELECT * FROM @job_execution_state


INSERT INTO @filtered_jobs
    SELECT sjv.job_id,
           sjv.date_created,
           sjv.date_modified,
           ISNULL(jes.execution_job_status, 4), -- Will be NULL if the job is non-local or is not in @job_execution_state (NOTE: 4 = STATE_IDLE)
           CASE ISNULL(jes.execution_step_id, 0)
             WHEN 0 THEN NULL                   -- Will be NULL if the job is non-local or is not in @job_execution_state
             ELSE CONVERT(NVARCHAR, jes.execution_step_id) + N' (' + jes.execution_step_name + N')'
           END,
           jes.execution_retry_attempt,         -- Will be NULL if the job is non-local or is not in @job_execution_state
           0,  -- last_run_date placeholder    (we'll fix it up in step 3.3)
           0,  -- last_run_time placeholder    (we'll fix it up in step 3.3)
           5,  -- last_run_outcome placeholder (we'll fix it up in step 3.3 - NOTE: We use 5 just in case there are no jobservers for the job)
           jes.next_run_date,                   -- Will be NULL if the job is non-local or is not in @job_execution_state
           jes.next_run_time,                   -- Will be NULL if the job is non-local or is not in @job_execution_state
           jes.next_run_schedule_id,            -- Will be NULL if the job is non-local or is not in @job_execution_state
           0,   -- type placeholder             (we'll fix it up in step 3.4)
		   ISNULL(jes.running,0)
    FROM msdb.dbo.sysjobs_view                sjv
         LEFT OUTER JOIN @job_execution_state jes ON (sjv.job_id = jes.job_id)
		 /*
    WHERE ((@schedule_id IS NULL)
      OR   (EXISTS(SELECT * 
                 FROM sysjobschedules as js
                 WHERE (sjv.job_id = js.job_id)
                   AND (js.schedule_id = @schedule_id))))
				   */

UPDATE @filtered_jobs
  SET current_execution_status = NULL
  WHERE (current_execution_status = 4)
    AND (job_id IN (SELECT job_id
                    FROM msdb.dbo.sysjobservers
                    WHERE (server_id <> 0)))

  -- Step 3.2: Check that if the user asked to see idle jobs that we still have some.
  --           If we don't have any then the query should return no rows.
  
 UPDATE @filtered_jobs
    SET last_run_date = sjs.last_run_date,
        last_run_time = sjs.last_run_time,
        last_run_outcome = sjs.last_run_outcome
    FROM @filtered_jobs         fj,
         msdb.dbo.sysjobservers sjs
    WHERE (fj.job_id = sjs.job_id)
  


  SELECT originating_server, sjv.name,sjv.enabled,fj.last_run_date, fj.last_run_time, fj.running, fj.last_run_outcome,
	CASE 
		WHEN fj.last_run_outcome = 0 THEN 'Failed'
        WHEN fj.last_run_outcome = 1 THEN 'Success'
        WHEN fj.last_run_outcome = 3 THEN 'Cancelled'
        ELSE 'Unknown'
	end AS Last_Run_OutCome_Description,
    next_run_date = ISNULL(fj.next_run_date, 0),                                 -- This column will be NULL if the job is non-local
    next_run_time = ISNULL(fj.next_run_time, 0),  
    current_execution_status = ISNULL(fj.current_execution_status, 0),           -- This column will be NULL if the job is non-local
    current_execution_step = ISNULL(fj.current_execution_step, N'0 ' + FORMATMESSAGE(14205)) -- This column will be NULL if the job is non-local

  FROM @filtered_jobs fj
       LEFT OUTER JOIN msdb.dbo.sysjobs_view  sjv ON (fj.job_id = sjv.job_id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so1 ON (sjv.notify_email_operator_id = so1.id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so2 ON (sjv.notify_netsend_operator_id = so2.id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so3 ON (sjv.notify_page_operator_id = so3.id)
       LEFT OUTER JOIN msdb.dbo.syscategories sc  ON (sjv.category_id = sc.category_id)
  -- ORDER BY sjv.job_id
  WHERE sjv.enabled=1
  AND sjv.name like '%full%backup%'
  AND (sjv.name <> 'DBA_copy_fullbackup_to_DR' AND sjv.name <> 'Temp_full_backup.Subplan_1')
  ORDER by name

  /*
  SELECT sjv.job_id,
         originating_server, 
         sjv.name,
         sjv.enabled,
         sjv.description,
         sjv.start_step_id,
         category = ISNULL(sc.name, FORMATMESSAGE(14205)),
         --owner = dbo.SQLAGENT_SUSER_SNAME(sjv.owner_sid),
         sjv.notify_level_eventlog,
         sjv.notify_level_email,
         sjv.notify_level_netsend,
         sjv.notify_level_page,
         notify_email_operator   = ISNULL(so1.name, FORMATMESSAGE(14205)),
         notify_netsend_operator = ISNULL(so2.name, FORMATMESSAGE(14205)),
         notify_page_operator    = ISNULL(so3.name, FORMATMESSAGE(14205)),
         sjv.delete_level,
         sjv.date_created,
         sjv.date_modified,
         sjv.version_number,
         fj.last_run_date,
         fj.last_run_time,
		 fj.running,
         fj.last_run_outcome,

		 CASE 
                WHEN fj.last_run_outcome = 0 THEN 'Failed'
                WHEN fj.last_run_outcome = 1 THEN 'Success'
                WHEN fj.last_run_outcome = 3 THEN 'Cancelled'
                ELSE 'Unknown'
            end AS Last_Run_OutCome_Description,

         next_run_date = ISNULL(fj.next_run_date, 0),                                 -- This column will be NULL if the job is non-local
         next_run_time = ISNULL(fj.next_run_time, 0),                                 -- This column will be NULL if the job is non-local
         next_run_schedule_id = ISNULL(fj.next_run_schedule_id, 0),                   -- This column will be NULL if the job is non-local
         current_execution_status = ISNULL(fj.current_execution_status, 0),           -- This column will be NULL if the job is non-local
         current_execution_step = ISNULL(fj.current_execution_step, N'0 ' + FORMATMESSAGE(14205)), -- This column will be NULL if the job is non-local
         current_retry_attempt = ISNULL(fj.current_retry_attempt, 0),                 -- This column will be NULL if the job is non-local
         has_step = (SELECT COUNT(*)
                     FROM msdb.dbo.sysjobsteps sjst
                     WHERE (sjst.job_id = sjv.job_id)),
         has_schedule = (SELECT COUNT(*)
                         FROM msdb.dbo.sysjobschedules sjsch
                         WHERE (sjsch.job_id = sjv.job_id)),
         has_target = (SELECT COUNT(*)
                       FROM msdb.dbo.sysjobservers sjs
                       WHERE (sjs.job_id = sjv.job_id)),
         type = fj.type
  FROM @filtered_jobs                         fj
       LEFT OUTER JOIN msdb.dbo.sysjobs_view  sjv ON (fj.job_id = sjv.job_id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so1 ON (sjv.notify_email_operator_id = so1.id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so2 ON (sjv.notify_netsend_operator_id = so2.id)
       LEFT OUTER JOIN msdb.dbo.sysoperators  so3 ON (sjv.notify_page_operator_id = so3.id)
       LEFT OUTER JOIN msdb.dbo.syscategories sc  ON (sjv.category_id = sc.category_id)
  -- ORDER BY sjv.job_id
  WHERE sjv.enabled=1
  AND sjv.name like '%full%backup%'
  ORDER by name
  */



  --SELECT * FROM @filtered_jobs
  
  
  
  SELECT
   session_id as SPID, command, a.text AS Query, start_time, percent_complete,
   dateadd(second,estimated_completion_time/1000, getdate()) as estimated_completion_time
FROM sys.dm_exec_requests r
   CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
WHERE r.command in ('BACKUP DATABASE','RESTORE DATABASE')

                                      BACKUP CHECK

SELECT r.session_id,r.command,CONVERT(NUMERIC(6,2),r.percent_complete)
AS [Percent Complete],CONVERT(VARCHAR(20),DATEADD(ms,r.estimated_completion_time,GetDate()),20) AS [ETA Completion Time],
CONVERT(NUMERIC(10,2),r.total_elapsed_time/1000.0/60.0) AS [Elapsed Min],
CONVERT(NUMERIC(10,2),r.estimated_completion_time/1000.0/60.0) AS [ETA Min],
CONVERT(NUMERIC(10,2),r.estimated_completion_time/1000.0/60.0/60.0) AS [ETA Hours],
CONVERT(VARCHAR(1000),(SELECT SUBSTRING(text,r.statement_start_offset/2,
CASE WHEN r.statement_end_offset = -1 THEN 1000 ELSE (r.statement_end_offset-r.statement_start_offset)/2 END)
FROM sys.dm_exec_sql_text(sql_handle))) AS [SQL]
FROM sys.dm_exec_requests r WHERE command IN ('RESTORE DATABASE','BACKUP DATABASE')



To check backup history 
use msdb 



backup drives
Virtual backup= commvault
               =Rubrik
			   
			   
drive names=native backup									 


                              BACKUP CHECK
							  
							  SELECT  
   A.[Server],   
   B.logical_device_name,  
   B.physical_device_name,   
   B.backupset_name, 
   B.description 
FROM 
   ( 
   SELECT   
      CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, 
      msdb.dbo.backupset.database_name,  
      MAX(msdb.dbo.backupset.backup_finish_date) AS last_db_backup_date 
   FROM 
      msdb.dbo.backupmediafamily  
      INNER JOIN msdb.dbo.backupset ON msdb.dbo.backupmediafamily.media_set_id = msdb.dbo.backupset.media_set_id  
   WHERE 
      msdb..backupset.type = 'D' 
   GROUP BY 
      msdb.dbo.backupset.database_name  
   ) AS A 
   LEFT JOIN  
   ( 
   SELECT   
      CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, , 
      msdb.dbo.backupset.backup_size,  
      msdb.dbo.backupmediafamily.logical_device_name,  
      msdb.dbo.backupmediafamily.physical_device_name,   
      msdb.dbo.backupset.name AS backupset_name, 
      msdb.dbo.backupset.description 
   FROM 
      msdb.dbo.backupmediafamily  
      INNER JOIN msdb.dbo.backupset ON msdb.dbo.backupmediafamily.media_set_id = msdb.dbo.backupset.media_set_id  
   WHERE 
      msdb..backupset.type = 'D' 
   ) AS B 
   ON A.[server] = B.[server] AND A.[database_name] = B.[database_name] AND A.[last_db_backup_date] = B.[backup_finish_date] 
ORDER BY  
   A.database_name 
   
                                                     Blockings CHECKING
													 
													 
													 
													  Blocking
													  -------

SET NOCOUNT ON
GO
SELECT SPID, BLOCKED, REPLACE (REPLACE (T.TEXT, CHAR(10), ' '), CHAR (13), ' ' ) AS BATCH
INTO #T
FROM sys.sysprocesses R CROSS APPLY sys.dm_exec_sql_text(R.SQL_HANDLE) T
GO
WITH BLOCKERS (SPID, BLOCKED, LEVEL, BATCH)
AS
(
SELECT SPID,
BLOCKED,
CAST (REPLICATE ('0', 4-LEN (CAST (SPID AS VARCHAR))) + CAST (SPID AS VARCHAR) AS VARCHAR (1000)) AS LEVEL,
BATCH FROM #T R
WHERE (BLOCKED = 0 OR BLOCKED = SPID)
AND EXISTS (SELECT * FROM #T R2 WHERE R2.BLOCKED = R.SPID AND R2.BLOCKED <> R2.SPID)
UNION ALL
SELECT R.SPID,
R.BLOCKED,
CAST (BLOCKERS.LEVEL + RIGHT (CAST ((1000 + R.SPID) AS VARCHAR (100)), 4) AS VARCHAR (1000)) AS LEVEL,
R.BATCH FROM #T AS R
INNER JOIN BLOCKERS ON R.BLOCKED = BLOCKERS.SPID WHERE R.BLOCKED > 0 AND R.BLOCKED <> R.SPID
)
SELECT N'    ' + REPLICATE (N'|         ', LEN (LEVEL)/4 - 1) +
CASE WHEN (LEN(LEVEL)/4 - 1) = 0
THEN 'HEAD -  '
ELSE '|------  ' END
+ CAST (SPID AS NVARCHAR (10)) + N' ' + BATCH AS BLOCKING_TREE
FROM BLOCKERS ORDER BY LEVEL ASC
GO
DROP TABLE #T
GO


/*

sp_whoisactive @sort_order='[CPU] DESC'

sp_whoisactive

sp_who4 active

sp_current_process

select * from sys.sysprocesses where spid=69  

kill 164 with statusonly
  
sp_who4 493

sp_current_process 493

select * from sys.sysprocesses where blocked <> 0 and blocked <> spid

select * from sys.sysprocesses where waitresource like '%100%'

select count(*) from sys.sysprocesses where blocked <> 0 and blocked <> spid

select count(*) from sys.sysprocesses where status <> 'sleeping'

select count(*) from sys.sysprocesses

select * from msdb.dbo.sysjobs where job_id=0x011F93FB30A47F45B0C4AC6C4B7691B2

select percent_complete, blocking_session_id, * from sys.dm_exec_requests where session_id in (321)

*/



Blocking
-----------

/*Blockings*/
EXEC sp_WhoIsActive
    @find_block_leaders = 1,
    @output_column_list ='[session_id][start_time][login_name][blocked_session_count][blocking_session_id]
                  [status][program_name]
                 [database_name][sql_text][host_name][open_tran_count]',
    @sort_order = '[blocked_session_count] DESC'




select * from sysprocesses where blocked >0



                                      CHECK DRIVE SPACE


 select collection_time,database_name,file_type,file_name,file_location,file_size_mb,used_space_mb,free_space_mb from [DBA_Admin].[dbo].[db_size_daily_report]
where file_drive='D:\' and CONVERT(DATE, collection_time)= CONVERT(DATE, getdate())
order by file_size_mb desc




                              CHECK ENCRYPTION STATUS
							  
							  
USE [master];
-- Check Encryption Status
SELECT DB_NAME(database_id) as dbName, encryption_state_desc, percent_complete,
		encryptor_thumbprint,
		[encryption_key_create_date] = regenerate_date,
		encryption_scan_state_desc,
		*
FROM sys.dm_database_encryption_keys 
WHERE database_id > 4
--and DB_NAME(database_id) in ('')
GO



DATA FILE SHRINK

USE MCDX
GO
Declare @cur int
Declare @inc int

set @cur = 464801    -- Current Total Space
set @inc = 1024     -- Pack by cursor
While @cur > 463000  -- Minimum Shrink UPTO
Begin

	Set @cur = @cur - @inc

	DBCC SHRINKFILE (MCDX_Data,@cur)

	Print @cur

End



DB SIZE

WITH CTE AS
(
SELECT sd.NAME
,SUM((CAST(smf.size AS FLOAT) * 8096) / (1024 * 1024)) AS SizeMB
,SUM((CAST(smf.size AS FLOAT) * 8096) / (1024 * 1024 * 1024)) AS SizeGB
FROM sys.databases sd
	INNER JOIN sys.master_files smf ON sd.database_id = smf.database_id
WHERE smf.type_desc IN ('ROWS','LOG') AND sd.name <> 'tempdb'
GROUP BY sd.Name
--ORDER BY sd.Name 
)
SELECT SUM(SizeGB) AS AllDBSize_GB  FROM CTE









                           TO CHECK DOMAIN OF THE SERVER

SELECT DEFAULT_DOMAIN() [DomainName]

select@@server name

select @@version

SELECT HOST_NAME()



HIGH CONNECTIONS

Number of sessions by particular user
===============================

SELECT DB_NAME(dbid) AS DBName,
COUNT(dbid) AS NumberOfConnections,hostname,
loginame
FROM   sys.sysprocesses
GROUP BY dbid, loginame,hostname
ORDER BY COUNT(dbid) desc
select count(*) as sessions,
    s.host_name,
    s.host_process_id,
    s.program_name,
    db_name(s.database_id) as database_name
from sys.dm_exec_sessions s
where is_user_process = 1
group by host_name, host_process_id, program_name, database_id
order by count(*) desc;
SELECT blocking_session_id AS BlockingSessionID, session_id AS VictimSessionID,
[text] AS VictimQuery, wait_time/1000/60 AS WaitDurationmins
FROM sys.dm_exec_requests
CROSS APPLY sys.dm_exec_sql_text([sql_handle])
WHERE blocking_session_id > 0
order by WaitDurationmins desc
select plan_handle,query_plan,TEXT from sys.dm_exec_requests
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
where session_id=103
select b.spid,b.loginame,b.hostname,a.client_net_address from sys.dm_exec_connections a,sys.sysprocesses b
where a.session_id=b.spid
and b.hostname='ABMUMFAP46'



SELECT * FROM sys.dm_exec_sessions


SELECT * FROM sys.dm_exec_sessions where status='sleeping';


-CONNECTIONS

SELECT cOUNT(*),LOGIN_NAME,host_name FROM SYS.dm_exec_sessions
WHERE is_user_process=1
GROUP BY LOGIN_NAME,host_name
ORDER BY cOUNT(*) DESC


--CONNECTIONS QUERY

SP_WHOISACTIVE @filter_type='host',@filter='ABMUBOAP186'


--CONNECTIONS BY HOST


select * FROM SYS.dm_exec_sessions

WHERE is_user_process=1 and host_name='ABMUBOAP186'

select * from sys.dm_exec_connections


JOB ENABLED OR DISabled


exec msdb..sp_update_job @job_name = 'NCX FUTURES AUTO PROCESS SETTLEMENT', @enabled = 0 --Disable
exec msdb..sp_update_job @job_name = 'NCX FUTURES AUTO PROCESS SETTLEMENT', @enabled = 1 --Enable


                 job query

SELECT * FROM msdb.dbo.sysjobs_view;

latency check



SELECT
   [ReadLatency] =
        CASE WHEN [num_of_reads] = 0
            THEN 0 ELSE ([io_stall_read_ms] / [num_of_reads]) END,
   [WriteLatency] =
        CASE WHEN [num_of_writes] = 0
            THEN 0 ELSE ([io_stall_write_ms] / [num_of_writes]) END,
   [Latency] =
        CASE WHEN ([num_of_reads] = 0 AND [num_of_writes] = 0)
            THEN 0 ELSE ([io_stall] / ([num_of_reads] + [num_of_writes])) END,
   [Latency Desc] = 
         CASE 
            WHEN ([num_of_reads] = 0 AND [num_of_writes] = 0) THEN 'N/A' 
            ELSE 
               CASE WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 2 THEN 'Excellent'
                    WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 6 THEN 'Very good'
                    WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 11 THEN 'Good'
                    WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 21 THEN 'Poor'
                    WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 101 THEN 'Bad'
                    WHEN ([io_stall] / ([num_of_reads] + [num_of_writes])) < 501 THEN 'Yikes!'
               ELSE 'YIKES!!'
               END 
         END, 
   [AvgBPerRead] =
        CASE WHEN [num_of_reads] = 0
            THEN 0 ELSE ([num_of_bytes_read] / [num_of_reads]) END,
   [AvgBPerWrite] =
        CASE WHEN [num_of_writes] = 0
            THEN 0 ELSE ([num_of_bytes_written] / [num_of_writes]) END,
   [AvgBPerTransfer] =
        CASE WHEN ([num_of_reads] = 0 AND [num_of_writes] = 0)
            THEN 0 ELSE
                (([num_of_bytes_read] + [num_of_bytes_written]) /
                ([num_of_reads] + [num_of_writes])) END,
   LEFT ([mf].[physical_name], 2) AS [Drive],
   DB_NAME ([vfs].[database_id]) AS [DB],
   [mf].[physical_name]
FROM
   sys.dm_io_virtual_file_stats (NULL,NULL) AS [vfs]
   JOIN sys.master_files AS [mf]
   ON [vfs].[database_id] = [mf].[database_id]
      AND [vfs].[file_id] = [mf].[file_id]
-- WHERE [vfs].[file_id] = 2 -- log files
ORDER BY [Latency] DESC
-- ORDER BY [ReadLatency] DESC
-- ORDER BY [WriteLatency] DESC;
GO



what is latency query

it's working only angel or all servers using tis command




SELECT   LEFT(mf.physical_name,100),  
         ReadLatency = CASE WHEN num_of_reads = 0 THEN 0 ELSE (io_stall_read_ms / num_of_reads) END, 
         WriteLatency = CASE WHEN num_of_writes = 0 THEN 0 ELSE (io_stall_write_ms / num_of_writes) END, 
         AvgLatency =  CASE WHEN (num_of_reads = 0 AND num_of_writes = 0) THEN 0 
                        ELSE (io_stall / (num_of_reads + num_of_writes)) END,
         LatencyAssessment = CASE WHEN (num_of_reads = 0 AND num_of_writes = 0) THEN 'No data' ELSE 
               CASE WHEN (io_stall / (num_of_reads + num_of_writes)) < 2 THEN 'Excellent' 
                    WHEN (io_stall / (num_of_reads + num_of_writes)) BETWEEN 2 AND 5 THEN 'Very good' 
                    WHEN (io_stall / (num_of_reads + num_of_writes)) BETWEEN 6 AND 15 THEN 'Good' 
                    WHEN (io_stall / (num_of_reads + num_of_writes)) BETWEEN 16 AND 100 THEN 'Poor' 
                    WHEN (io_stall / (num_of_reads + num_of_writes)) BETWEEN 100 AND 500 THEN  'Bad' 
                    ELSE 'Deplorable' END  END, 
         [Avg KBs/Transfer] =  CASE WHEN (num_of_reads = 0 AND num_of_writes = 0) THEN 0 
                    ELSE ((([num_of_bytes_read] + [num_of_bytes_written]) / (num_of_reads + num_of_writes)) / 1024) END, 
         LEFT (mf.physical_name, 2) AS Volume, 
         LEFT(DB_NAME (vfs.database_id),32) AS [Database Name]
       FROM sys.dm_io_virtual_file_stats (NULL,NULL) AS vfs  
       JOIN sys.master_files AS mf ON vfs.database_id = mf.database_id 
         AND vfs.file_id = mf.file_id 
       ORDER BY AvgLatency DESC
	   
	   
	 Second query 
	 
	   SELECT TOP (5) DB_NAME (a.database_id) AS dbname,
  a.io_stall / NULLIF (a.num_of_reads + a.num_of_writes, 0) AS average_tot_latency,
  Round ((a.size_on_disk_bytes / square (1024.0)), 1) AS size_mb,
  b.physical_name AS [fileName]
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS a,
  sys.master_files AS b
WHERE a.database_id = b.database_id
  AND a.FILE_ID = b.FILE_ID
ORDER BY average_tot_latency DESC




login and user creation


User create

use Sales_BI
go
CREATE USER [E83122] for login [E83122];


grant select on TB_RNE_Voucher  to [E83122]

LOGIN

USE [master]
GO
CREATE LOGIN [] FROM WINDOWS WITH DEFAULT_DATABASE=[master]
GO



to find login exist


select * from [dbo].[User_Login_Details] where Login_id='E85802'



To provide permissions on all databases
FOR SQL 2012 onwards
========================================================================================================================

USE [master]
GO
CREATE LOGIN [E84570] WITH PASSWORD=N'b@F0HFLaTE8S%ysvdqwe', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO

DECLARE @dbName VARCHAR(100)
DECLARE @sSql NVARCHAR(4000)
DECLARE @Login VARCHAR(50)

SET @Login = 'E84570'

DECLARE curDB CURSOR FOR SELECT Name FROM sys.databases WHERE database_id > 4 AND state_desc = 'ONLINE' AND Name NOT LIKE 'ReportServer%'
OPEN curDB
FETCH NEXT FROM curDB INTO @dbName
WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @sSql = 'USE [' + @dbName + '] 
					GO
					CREATE USER [' + @Login + '] FOR LOGIN [' + @Login + ']
					GO
					ALTER ROLE [db_datareader] ADD MEMBER [' + @Login + ']
					GO
					--ALTER ROLE [db_datawriter] ADD MEMBER [' + @Login + ']
					--GO
					--ALTER ROLE [db_ddladmin] ADD MEMBER [' + @Login + ']
					--GO
					--ALTER ROLE [db_owner] ADD MEMBER [' + @Login + ']
					--GO
					GRANT EXECUTE TO [' + @Login + ']
					--GO'

		PRINT @sSql

		FETCH NEXT FROM curDB INTO @dbName

	END

CLOSE curDB
DEALLOCATE curDB


========================================================================================================================
For SQL 2008
========================================================================================================================

USE [master]
GO
CREATE LOGIN [E83184] WITH PASSWORD=N'njksyijg %$12hy&%jl34', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO

DECLARE @dbName VARCHAR(100)
DECLARE @sSql NVARCHAR(4000)
DECLARE @Login VARCHAR(50)

SET @Login = 'E83184'

DECLARE curDB CURSOR FOR SELECT Name FROM sys.databases WHERE database_id > 4 AND state_desc = 'ONLINE' AND Name NOT LIKE 'ReportServer%'
OPEN curDB
FETCH NEXT FROM curDB INTO @dbName
WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @sSql = 'USE [' + @dbName + '] 
					GO
					CREATE USER [' + @Login + '] FOR LOGIN [' + @Login + ']
					GO
					EXEC sp_addrolemember N''db_datareader'', N''' + @Login + '''
					GO'
		PRINT @sSql

		FETCH NEXT FROM curDB INTO @dbName

	END

CLOSE curDB
DEALLOCATE curDB




TO CHECK DOMAIN OF THE SERVER

SELECT DEFAULT_DOMAIN() [DomainName]



RETURN 0

Blockings occurs if a particular query is running in single SP ID then need to add RETURN 0





                       login database access


USE [master]
GO
CREATE LOGIN [abcd] WITH PASSWORD=N'w@YVtcb3JW9u9j#J', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO
use [DPC];
GO
use [master];
GO
USE [DPC]
GO
CREATE USER [abcd] FOR LOGIN [abcd]
GO
USE [DPC]
GO
ALTER ROLE [db_datareader] ADD MEMBER [abcd]
GO
USE [DPC]
GO
ALTER ROLE [db_datawriter] ADD MEMBER [abcd]
GO
USE [DPC]
GO
ALTER ROLE [db_ddladmin] ADD MEMBER [abcd]
GO

                   login insert 
				   
INSERT INTO [dbo].[User_Login_Details]
           ([Login_ID]
           ,[Password]
           ,[Login_Name]
          )
     VALUES
           ('intern_jainam',
           ':>V-/F!T"&9~gEB5Gbp@',     fill the details here
           'Jainam Dhami'
           )
GO

--------------------------------------------------------------------------------------------------

select * from [dbo].[User_Login_Details] where Login_name like '%Utkarsh%' 
	
select * from [dbo].[User_Login_Details] where Login_id='E83176'



Add in DBA_ADMIN



logins

USE [master]
GO
CREATE LOGIN [inhouse] WITH PASSWORD=N'inh6014', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
GO
use [master];
GO
USE [uploader-db]
GO
CREATE USER [inhouse] FOR LOGIN [inhouse]
GO
USE [uploader-db]
GO
ALTER ROLE [db_ddladmin] ADD MEMBER [inhouse]
GO



log space checking

dbcc sqlperf(logspace)


                                 LOng running query	


	 
SELECT  ltrim(rtrim(P.SPID)) spid,    
 --text,    
 [Currently_Running_Query] = SUBSTRING (QT.text,ER.statement_start_offset/2,                
 (CASE WHEN ER.statement_end_offset = -1 THEN LEN(CONVERT(NVARCHAR(MAX), QT.text)) * 2                
 ELSE ER.statement_end_offset END - ER.statement_start_offset)/2),    
 [Parent_Query] = QT.text,    
 Blocked_By = ltrim(rtrim(P.Blocked)), 
 P.waitresource,  
 P.login_time,
 P.last_batch,
 ltrim(rtrim(P.Status)) STATUS,    
 ltrim(rtrim(P.LogiName))Loginname,    
 ltrim(rtrim(P.HostName))HostName,    
 Program = coalesce('Job Name: ' + J.name, P.program_name),  
 object_name(QT.objectid,QT.dbid) OBJ_name,  
 DBName = ltrim(rtrim(db_name(P.dbid))),     
 Command = ltrim(rtrim(P.cmd)),    
 CPUTime = ltrim(rtrim(P.cpu)),    
 DiskIO = ltrim(rtrim(P.physical_io)),    
 --LastBatch = ltrim(rtrim(p.Last_Batch)),    
 ltrim(rtrim(P.WaitTime))waittime,       
 ltrim(rtrim(P.LastWaitType))LastWaitType,       
       
 [Server] = ltrim(rtrim(@@SERVERNAME)),       
 [Duration] = datediff(second, P.last_batch, getdate())     
FROM     
  msdb.dbo.sysjobs J     
 RIGHT OUTER JOIN master.dbo.sysprocesses P  with (NOLOCK)    
 ON SUBSTRING(P.PROGRAM_NAME,32,32) = SUBSTRING(master.dbo.fn_varbintohexstr(J.job_id),3,100)     
 LEFT OUTER JOIN sys.dm_exec_requests ER    with (NOLOCK)    
 ON ER.SESSION_ID = P.SPID     
 CROSS APPLY sys.dm_exec_sql_text(P.SQL_HANDLE) QT  --with (NOLOCK)    
 WHERE --P.SPID > 50     
 --AND DATEDIFF(MI,LAST_BATCH,GETDATE())>=10    
  P.SPID <> @@SPID     
 AND KPID <> 0    
ORDER BY P.SPID    
    


ms db data delete

USE MSDB
GO
SELECT MIN(sent_date), MAX(sent_date) FROM sysmail_allitems


DECLARE @StartTime DATETIME
DECLARE @sSql NVARCHAR(250)
SET @StartTime = '2023-03-14 23:56:18.000'

WHILE @StartTime < '2023-03-31 23:59:18.000'
	BEGIN
		SET @sSql = 'sysmail_delete_mailitems_sp @sent_before = ''' + CONVERT(VARCHAR(19), @StartTime, 121) + ''''
		EXEC sp_ExecuteSQL @sSql
		PRINT @sSql
		SET @StartTime = DATEADD(s, 300, @StartTime)

	END


                            query detailes time_started
SELECT * FROM DBA_Admin.dbo.WhoIsActive
WHERE Collection_Time BETWEEN '2023-06-06 04:40:00.000' AND '2023-06-06 05:30:00.000'	



							  
