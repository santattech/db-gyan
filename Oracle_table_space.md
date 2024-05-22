### Create tablespace

First. Need to create a folder (eg: 'C:\oracle_data\projectname_local_nolog\) inside 'C:\oracle_data\’, then run the following query. 

`CREATE TABLESPACE XYZ_TAB01_NOLOG DATAFILE 'C:\oracle_data\projectname_local_nolog\datafile.dat' SIZE 1024M AUTOEXTEND ON MAXSIZE 2048M; `

### Move a table to a different tablespace

`ALTER TABLE TBL_TRACK MOVE TABLESPACE XYZ_TAB01_NOLOG;`

### Move a table with partition to a different partition, you need to move partition by partition
`ALTER TABLE TBL_TRACK MOVE PARTITION TBL_TRACK_0 TABLESPACE XYZ_TAB01_NOLOG;`

### Error handling: 
  Insufficient errors on partition while running SQL insert query
  Error starting at line : 1 in command -
  
```
Error report -
ORA-01031: insufficient privileges
01031. 00000 -  "insufficient privileges"
*Cause:    An attempt was made to perform a database operation without
           the necessary privileges.
*Action:   Ask your database administrator or designated security
           administrator to grant you the necessary privileges
```

`ALTER USER XYZ_TEST quota 100M on XYZ_TAB01_NOLOG;`

### Create an index in a particular tablespace
`CREATE INDEX I_TBL_TRACK_DATENUM_STARTTIME ON TBL_TRACK (DATE_NUM, STARTTIME) NOLOGGING TABLESPACE #{tablespace_name};`

### Create table in nologging tablespace 
```
CREATE TABLE TBL_XYZ_POSITION (
       X NUMBER,
       Y NUMBER,
       BOX_ID NUMBER(4,0),
       IS_COLLECTING NUMBER(1,0) DEFAULT 0,
       DATATIME TIMESTAMP(6)
      ) NOLOGGING TABLESPACE #{tablespace_name};
```
### Check index tablespaces 
```
SELECT  index_name, tablespace_name FROM    all_indexes WHERE   table_name in ('TBL_TRACKONE', 'TBL_TRACK', 'TBL_OBU', 'TBL_SPEEDS', 'TBL_VOLUMES');
```
### Check more info
  Check more info on tablespace in the following tables
  - all_tables
  - user_tables
  - all_indexes
  - all_tab_partitions
