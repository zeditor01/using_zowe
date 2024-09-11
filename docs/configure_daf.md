[Back to Index](https://github.com/zeditor01/using_zowe/blob/main/README.md)

Last Update to this page: 4 September 2024

# Configure DDL and SQL Tuning Services in Db2 Administration Foundation

***Purpose of this Chapter***
Many of the functions of Db2 Administration Foundation will work after the basic installation, such as Catalog Navigation, SQL execution and Command Execution. Some facilities need a little further customisation work.

## Contents
1. Configure DDL Generation Services.
2. Configure SQL Tuning Services.


## 1. Configure DDL Generation Services.

Configuring DDL services is documented in the knowledge center [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-installing-db2-administration-foundation)

If you want to generate the DDL of an explored object, you must configure a Workload Manager (WLM) environment for your subsystem:

1. Create a WLM environment by using the template in JCLLIB(WLMPROC) that points to the load libraries in <HLQ>.SIZPLLIB.
2. Start the WLM by using the template in JCLLIB(WLMPROC) that points to the WLM you created in step 1.
3. While registering the subsystem, specify the name of the WLM in the Workload Manager environment field on the Configuration tab.


DBDGWLM1 was already defined, so I chose to call this PROC DBDGWLMZ

Edited PROC for DBDGWLMZ as follows
```
//DBDGWLMZ PROC APPLENV=DBDGWLMZ,                                        
//    DB2SSN=DBDG,NUMTCB=8                                               
//IEFPROC EXEC PGM=DSNX9WLM,REGION=0M,TIME=NOLIMIT,                      
//        PARM='&DB2SSN,&NUMTCB,&APPLENV'                                
//STEPLIB  DD DISP=SHR,DSN=ADBD10.SADBLLIB                               
//         DD DISP=SHR,DSN=DSND10.DGDG.RUNLIB.LOAD       DB2 RUNLIB      
//         DD DISP=SHR,DSN=DSND10.SDSNLOAD          DB2 SDSNLOAD         
//         DD DISP=SHR,DSN=DSND10.SDSNLOD2          DB2 SDSNLOD2         
//         DD DISP=SHR,DSN=CEE.SCEERUN             LANGUAGE ENVIRONMENT  
//UTPRINT  DD SYSOUT=*                                                   
//RNPRIN01 DD SYSOUT=*                                                   
//DSSPRINT DD SYSOUT=*                                                   
//SYSIN    DD UNIT=SYSDA,SPACE=(4000,(20,20),,,ROUND)                    
//SYSPRINT DD SYSOUT=*                                                   
//SYSOUT   DD SYSOUT=*                                                   
//CEEDUMP  DD SYSOUT=*                                                   
//*ADBMSGS  DD SYSOUT=*                                                  
//*ADBDIAG  DD SYSOUT=*                                                  
```

Create a WLM Environment 
```
**************************** Top of Data ******************************
                                                                       
Appl Environment Name . . DBDGENVZ                                     
Description . . . . . . . ZOWE_ENVIRONMENT                             
Subsystem type  . . . . . DB2                                          
Procedure name  . . . . . DBDGWLMZ                                     
Start parameters  . . . . DB2SSN=&IWMSSNM,APPLENV='DBDGENVZ'           
                                                                       
                                                                       
                                                                       
Starting of server address spaces for a subsystem instance:            
 Managed by WLM                                                        
                                                                       
*************************** Bottom of Data ****************************
```




## 2. Configure SQL Tuning Services.

Blah blah


