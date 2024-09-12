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


### 1.1 Define the WLM PROC for ZOWE

DBDGWLM1 was already defined, so I chose to call this PROC DBDGWLMZ

Edited PROC for DBDGWLMZ as follows, and copied it to ADCD.Z31A.PROCLIB where all the other DB2 WLM Procs reside.
```
//DBDGWLMZ PROC APPLENV=DBDGENVZ,                                         
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

### 1.2 Create a WLM Environment for it

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

Install the modified WLM service definition, by selecting the "install" option when exiting from WLM editing ISPF panels.

Then activate the WLM policy that includes this new service definition.
```
VARY WLM, POLICY=ETPBASE
```

And finally, activate the APPLENV and veify that it is available.
```
/V WLM,APPLENV=DBDGENVZ,RESUME
```

followed by

```
D WLM,APPLENV=DBDGENVZ                                  
IWM029I  06.21.27  WLM DISPLAY 481                      
  APPLICATION ENVIRONMENT NAME     STATE     STATE DATA 
  DBDGENVZ                         AVAILABLE            
  ATTRIBUTES: PROC=DBDGWLMZ SUBSYSTEM TYPE: DB2         
```



### 1.3 Modify the ZWEYAML to discover the tools

You may need to apply a PTF to provide tools discovery capability to Db2 Administration Tool [APAR PH55177](https://www.ibm.com/docs/en/db2admintool/13.1?topic=tool-enabling-product-discovery).
```
toolsDiscovery:
  enabled: true
  discoverySearchPaths:
  - "DSN:ADBD10.SHLOSAMP(ADB131P)"
```

Configure the statistics tab as per the [docco](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=foundation-configuring-statistics-tab)
```
toolsDiscovery:
  enabled: true
  discoverySearchPaths:
  - "DSN:HLQ.SADBSAMP(ADBDSCVP)"
```


## 2. Configure SQL Tuning Services.

Configuring SQL Tuning Services requires that you follow two sets of instructions
1. Setup the Tuning Services, as documented in the OQWT documentation [here](https://www.ibm.com/docs/en/dqwtfz/6.1?topic=installation-roadmaps)
2. Configure the UMS PARMLIB member (IZPDB2PM) to use the tuning services, as documented in UMS [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-configuring-ums-sql-tuning-services-db2)

```
tuningHost
    The address of the host where SQL Tuning Services is running.
tuningPort
    The port number that you must use when connecting to the host.
tuningThroughHttps
    The default and preferred value of this parameter is true. Set this value to false to enable the HTTP connection from Unified Management Server to the SQL Tuning Services server.
tuningDb2SecurityMechanismId
    The mechanism that you want to use for securing the connection to SQL Tuning Services. The default value of this parameter is 3.
maxInMemorySize
    This parameter is used to configure the response buffer size for SQL Tuning Services. Before configuring this value, consider the number of concurrent users and environments in your organization. The default value of this parameter is 10 MB.
tuningApplicationId
    The applname of the subsystem that is used to configure the repository database. This value must be consistent with the appl_id parameter set during the SQL Tuning Services configuration. This parameter is mandatory when MFA is enabled.
```

