---
layout:     post
title:      "Monitor RMAN backup and restore progress"
author:     "Iceberg"
header-img: "assets/images/header/yellowstone7.jpg"
catalog: true
tags:
  - Database
  - Oracle	
---

The following script will allow you to monitor progress of an RMAN backup or Restore from 4 different perspectives (channel, session wait events, datafiles, backuppieces). The script is run from SQLPlus and takes a date input value in the format ‘dd-mon-rr hh24:mi:ss’. The date supplied does not have to be precise and can be taken from the rman log of the job that is running.

```shell
REM 
REM Script to monitor rman backup/restore operations
REM To run from sqlplus:   @monitor '[dd-mon-rr hh24:mi:ss]' 
REM Example:  
--SQL>spool monitor.out
--SQL>@monitor '06-aug-12 16:38:03'
REM where [date] is the start time of your rman backup or restore job
REM Run monitor script periodically to confirm rman is progessing
REM 

alter session set nls_date_format='dd-mon-rr hh24:mi:ss';
set lines 1500
set pages 100
col CLI_INFO format a10
col spid format a5
col ch format a20
col seconds format 999999.99
col filename format a65
col bfc  format 9
col "% Complete" format 999.99
col event format a40
set numwidth 10

select sysdate from dual;

REM gv$session_longops (channel level)

prompt
prompt Channel progress - gv$session_longops:
prompt
select s.inst_id, o.sid, CLIENT_INFO ch, context, sofar, totalwork,
                    round(sofar/totalwork*100,2) "% Complete"
     FROM gv$session_longops o, gv$session s
     WHERE opname LIKE 'RMAN%'
     AND opname NOT LIKE '%aggregate%'
     AND o.sid=s.sid
     AND totalwork != 0
     AND sofar <> totalwork;

REM Check wait events (RMAN sessions) - this is for CURRENT waits only
REM use the following for 11G+
prompt
prompt Session progess - CURRENT wait events and time in wait so far:
prompt
select inst_id, sid, CLIENT_INFO ch, seq#, event, state, wait_time_micro/1000000 seconds
from gv$session where program like '%rman%' and
wait_time = 0 and
not action is null;

REM use the following for 10G
--select  inst_id, sid, CLIENT_INFO ch, seq#, event, state, seconds_in_wait secs
--from gv$session where program like '%rman%' and
--wait_time = 0 and
--not action is null;

REM gv$backup_async_io
prompt
prompt Disk (file and backuppiece) progress - includes tape backuppiece 
prompt if backup_tape_io_slaves=TRUE:
prompt
select s.inst_id, a.sid, CLIENT_INFO Ch, a.STATUS,
open_time, round(BYTES/1024/1024,2) "SOFAR Mb" , round(total_bytes/1024/1024,2)
TotMb, io_count,
round(BYTES/TOTAL_BYTES*100,2) "% Complete" , a.type, filename
from gv$backup_async_io a,  gv$session s
where not a.STATUS in ('UNKNOWN')
and a.sid=s.sid and open_time > to_date('&1', 'dd-mon-rr hh24:mi:ss') order by 2,7;

REM gv$backup_sync_io
prompt
prompt Tape backuppiece progress (only if backup_tape_io_slaves=FALSE):
prompt
select s.inst_id, a.sid, CLIENT_INFO Ch, filename, a.type, a.status, buffer_size bsz, buffer_count bfc,
open_time open, io_count
from gv$backup_sync_io a, gv$session s
where
a.sid=s.sid and
open_time > to_date('&1', 'dd-mon-rr hh24:mi:ss') ;
REM 
```

From RMAN log, you can see the follwoing timestamp.

*Starting restore at 16-DEC-2019 12:38:04*

You can run this script at any time if you suspect that rman is taking longer than expected - simply spool the results to a file and rerun the script periodically to check that the job is progressing.

Sample output:

```shell
$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Mon Dec 16 14:08:34 2019
Copyright (c) 1982, 2016, Oracle.  All rights reserved.
Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> @monitor "16-DEC-2019 12:38:04"
Session altered.


SYSDATE
16-dec-19 14:08:45


Channel progress - gv$session_longops:
no rows selected

Session progess - CURRENT wait events and time in wait so far:

   INST_ID	  SID CH			 SEQ# EVENT				       STATE		      SECONDS

	 1	  217				35823 SQL*Net message from client	       WAITING		       115.75
	 1	  225 rman channel=ch00 	 3647 Backup: MML restore backup piece	       WAITING		       115.94
	 1	  229 rman channel=ch01 	32765 Backup: MML restore backup piece	       WAITING		       115.88
	 1	  233 rman channel=ch02 	51706 Backup: MML restore backup piece	       WAITING		       115.81
	 1	  237 rman channel=ch03 	62302 Backup: MML restore backup piece	       WAITING		       115.75

Disk (file and backuppiece) progress - includes tape backuppiece
if backup_tape_io_slaves=TRUE:

old   7: and a.sid=s.sid and open_time > to_date('&1', 'dd-mon-rr hh24:mi:ss') order by 2,7
new   7: and a.sid=s.sid and open_time > to_date('16-DEC-2019 12:38:04', 'dd-mon-rr hh24:mi:ss') order by 2,7

   INST_ID	  SID CH		   STATUS      OPEN_TIME	    SOFAR Mb	  TOTMB   IO_COUNT % Complete TYPE	FILENAME

	 1	  225 rman channel=ch00    FINISHED    16-dec-19 13:03:03	  31	     31 	 9     100.00 OUTPUT	+DATA/tpcc/iitem_0_0
	 1	  225 rman channel=ch00    FINISHED    16-dec-19 12:47:56      31900	  31900       7977     100.00 OUTPUT	+DATA/tpcc/hist_0_0
	 1	  229 rman channel=ch01    FINISHED    16-dec-19 12:57:30      30600	  30600       7652     100.00 OUTPUT	+DATA/tpcc/stok_0_6
	 1	  229 rman channel=ch01    FINISHED    16-dec-19 12:53:37      30890	  30890       7724     100.00 OUTPUT	+DATA/tpcc/ordr_0_9
	 1	  229 rman channel=ch01    FINISHED    16-dec-19 12:50:21      31280	  31280       7822     100.00 OUTPUT	+DATA/tpcc/cust_0_6
	 1	  233 rman channel=ch02    FINISHED    16-dec-19 13:02:39	  90	     90 	24     100.00 OUTPUT	+DATA/tpcc/ware_0_0
	 1	  233 rman channel=ch02    FINISHED    16-dec-19 13:01:49	 933	    933        235     100.00 OUTPUT	+DATA/tpcc/tpccaux
	 1	  233 rman channel=ch02    FINISHED    16-dec-19 12:58:16      30600	  30600       7652     100.00 OUTPUT	+DATA/tpcc/stok_0_11
	 1	  233 rman channel=ch02    FINISHED    16-dec-19 12:55:07      30890	  30890       7724     100.00 OUTPUT	+DATA/tpcc/ordr_0_14
	 1	  233 rman channel=ch02    FINISHED    16-dec-19 12:51:07      30890	  30890       7724     100.00 OUTPUT	+DATA/tpcc/ordr_0_4
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 13:02:48      33.21	  33.21 	10     100.00 OUTPUT	+DATA/tpcc/iware_0_0
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 13:01:59	 400	    400        102     100.00 OUTPUT	+DATA/tpcc/system_1
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 13:00:34    7490.45	7490.45       1874     100.00 OUTPUT	+DATA/tpcc/icust1_0_0
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 12:55:15      30600	  30600       7652     100.00 OUTPUT	+DATA/tpcc/stok_0_0
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 12:53:02      30890	  30890       7724     100.00 OUTPUT	+DATA/tpcc/ordr_0_8
	 1	  237 rman channel=ch03    FINISHED    16-dec-19 12:48:06      31280	  31280       7822     100.00 OUTPUT	+DATA/tpcc/cust_0_1

16 rows selected.

Tape backuppiece progress (only if backup_tape_io_slaves=FALSE):

old   6: open_time > to_date('&1', 'dd-mon-rr hh24:mi:ss')
new   6: open_time > to_date('16-DEC-2019 12:38:04', 'dd-mon-rr hh24:mi:ss')

   INST_ID	  SID CH		   FILENAME							     TYPE      STATUS		  BSZ BFC OPEN		       IO_COUNT

	 1	  225 rman channel=ch00    bk_dTPCC_ucuuj5dpp_s414_p1_t1026733881			     INPUT     FINISHED        262144	4 16-dec-19 12:47:56	  65392
	 1	  225 rman channel=ch00    bk_dTPCC_uehuj5udh_s465_p1_t1026750897			     INPUT     FINISHED        262144	4 16-dec-19 13:03:03	      8
	 1	  229 rman channel=ch01    bk_dTPCC_ud7uj5i8g_s423_p1_t1026738448			     INPUT     FINISHED        262144	4 16-dec-19 12:50:21	 116172
	 1	  229 rman channel=ch01    bk_dTPCC_udtuj5q1d_s445_p1_t1026746413			     INPUT     FINISHED        262144	4 16-dec-19 12:57:30	 113036
	 1	  229 rman channel=ch01    bk_dTPCC_udhuj5l9e_s433_p1_t1026741550			     INPUT     FINISHED        262144	4 16-dec-19 12:53:37	  63372
	 1	  233 rman channel=ch02    bk_dTPCC_ueeuj5u99_s462_p1_t1026750761			     INPUT     FINISHED        262144	4 16-dec-19 13:02:39	    312
	 1	  233 rman channel=ch02    bk_dTPCC_udmuj5mm0_s438_p1_t1026742976			     INPUT     FINISHED        262144	4 16-dec-19 12:55:07	  63776
	 1	  233 rman channel=ch02    bk_dTPCC_ue0uj5rik_s448_p1_t1026747988			     INPUT     FINISHED        262144	4 16-dec-19 12:58:16	 113444
	 1	  233 rman channel=ch02    bk_dTPCC_ueauj5u3a_s458_p1_t1026750570			     INPUT     FINISHED        262144	4 16-dec-19 13:01:49	   3220
	 1	  233 rman channel=ch02    bk_dTPCC_udauj5jb1_s426_p1_t1026739553			     INPUT     FINISHED        262144	4 16-dec-19 12:51:07	  64180
	 1	  237 rman channel=ch03    bk_dTPCC_ueguj5uc3_s464_p1_t1026750851			     INPUT     FINISHED        262144	4 16-dec-19 13:02:48	      0
	 1	  237 rman channel=ch03    bk_dTPCC_uecuj5u6f_s460_p1_t1026750671			     INPUT     FINISHED        262144	4 16-dec-19 13:01:59	   1184
	 1	  237 rman channel=ch03    bk_dTPCC_ue7uj5trl_s455_p1_t1026750325			     INPUT     FINISHED        262144	4 16-dec-19 13:00:34	  28188
	 1	  237 rman channel=ch03    bk_dTPCC_udnuj5mut_s439_p1_t1026743261			     INPUT     FINISHED        262144	4 16-dec-19 12:55:15	 113444
	 1	  237 rman channel=ch03    bk_dTPCC_udfuj5knj_s431_p1_t1026740979			     INPUT     FINISHED        262144	4 16-dec-19 12:53:02	  63372
	 1	  237 rman channel=ch03    bk_dTPCC_ud1uj5f5c_s417_p1_t1026735276			     INPUT     FINISHED        262144	4 16-dec-19 12:48:06	 115768

16 rows selected.
```

## References

* <https://www.thegeekdiary.com/script-to-monitor-rman-backup-and-restore-operations/>