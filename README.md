<a href="https://github.com/mortennobel/cpp-cheatsheet"><img align="right" src="https://camo.githubusercontent.com/38ef81f8aca64bb9a64448d0d70f1308ef5341ab/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f6461726b626c75655f3132313632312e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_darkblue_121621.png"></a>

# Oracle PL/SQL CHEATSHEET
The goal is to give a concise overview of basic, modern Oracle PL/SQL coding.

## EXCEPTION

```
--1.Default EXCEPTION : no_data_found, too_many_rows, others
DECLARE
...
BEGIN
...
EXCEPTION 
  --print log
  WHEN no_data_found THEN
    dbms_output.put_line('No data found error'); 
  --do nothing
  WHEN no_data_found OR too_many_rows THEN
    NULL;
  --print log + reraise error with "RAISE" or with a new error message by "raise_application_error()"
  WHEN OTHERS THEN  
    dbms_output.put_line('Raised exception: ERRCODE=' || SQLCODE || ', ERRMSG=' || SQLERRM); 
    raise_application_error(-20021, 'Other error');
    --RAISE;
END;
/


--2.Custom EXCEPTION
DECLARE
  -- ORA-00054: resource busy and acquire with NOWAIT specified
  RESOURCE_BUSY EXCEPTION;
  PRAGMA EXCEPTION_INIT(RESOURCE_BUSY, -54);

  vcLockedBy  t_locked_ref.createur%TYPE;
BEGIN
  IF l_lock_detected THEN
    RAISE RESOURCE_BUSY;
  END IF;
EXCEPTION
  WHEN RESOURCE_BUSY THEN
    BEGIN
      SELECT L.createur
           , L.sid
           , nvl(S.process, 'n/a')
           , nvl(S.program, 'n/a')
           , nvl(S.machine, 'n/a')
        INTO vcLockedBy
           , iSID
           , vcProcess
           , vcProgram
           , vcMachine
        FROM t_locked_ref L
           , v$session S
       WHERE L.sid        = S.sid (+)
         AND L.serial#    = S.serial# (+)
         AND L.logon_time = S.logon_time (+)
         AND L.type       = 'LIMIT'
         AND L.reference  = lock_manager.GetLockID( lCurLimit.base || '@' || lCurLimit.offset, 'LIMIT' );
    EXCEPTION
      WHEN NO_DATA_FOUND THEN
        vcLockedBy := 'UNKNOWN';
    END;
END;

```
