[Back to Index](https://github.com/zeditor01/using_zowe/blob/main/README.md)

Last Update to this page: 9 September 2024

# Deploying UMS

***Purpose of this Chapter***
Whilst UMS does have a couple of extra ZOWE Apps, it is primarily a pre-requisite for the Db2 tools. 
Hence, the scope of this chapter is limited to a worked example of installing and deploying UMS.

## Contents
1. Planning for UMS.
2. Installing the UMS product code.
3. Editing the ZWEYAML parmlib file.
4. Installing the UMS Instance.
5. Configuring the UMS instance.
6. Operating UMS.
7. Using the UMS Base Apps.
8. Subsequent Upgrades.


## 1. Planning for ZOWE.

UMS documentation

UMS architecture

RACF

This Worked Example

## 2. Installing the ZOWE product code.

Creating and Mounting a ZFS

Installing / Unpacking the ZOWE product COde

## 3. Editing the ZWEYAML parmlib file

Section 1

Section 2

hint right click - open in new tab

[ZWEYAML here](https://github.com/zeditor01/using_zowe/blob/main/samples/ZWEYAML.TXT)

[IZPDB2PM here](https://github.com/zeditor01/using_zowe/blob/main/samples/IZPDB2PM.TXT)

[IZPDAFPM here](https://github.com/zeditor01/using_zowe/blob/main/samples/IZPDAFPM.TXT)



## 4. Installing the UMS instance.

Invoking the Installer

## 5. Configuring the UMS instance.

Invoking the Configuration

## 6. Operating UMS.

PARMLIB

Automated Operations

## 7. Using the UMS Base Apps.

JES Explorer

Editor

TN3270

## 8. Subsequent Upgrades.

Parallel ZOWE instances

Upgrading a single ZOWE instance



```
SMPE install UMS and DAF.

/usr/lpp/IBM/izp/v1r2m0/bin
/usr/lpp/IBM/afx/v1r2m0/bin


Stop ZOWE
Stop ZOWE Cross Memory Server
    
Copy SIZPSAMP to another data set of the same attributes IZP.CUST.SAMPLIB ( members  IZPALOPL,  IZPCPYML, IZPCPYM2, IZPGENER, IZPMIGRA )

Edit and submit IZP.CUST.SAMPLIB(IZPALOPL) ... Allocates IZP.CUST.PARMLIB
   
Edit and submit IZP.CUST.SAMPLIB(IZPCPYML) ... Creates the ZWEYAML default PARMLIB member (to be edited).

(Migration Only) IZPMIGRA Migrates current values from version 1.1 to version 1.2.

Edit IZP.CUST.PARMLIB(ZWEYAML) 
- experiences
- useSAFOnly: true
- dba authentication PASSWORD
- profileQualifier: S0W1 
- dbaUser: IZPDBA
- token: IZPTOK
- truststore location: "////ZWESVUSR/ZoweKeyring"
- keystore location: "////ZWESVUSR/ZoweKeyring" and alias: "zowes0w1"
- profilePrefix:        
  # Super Role        
  super: IZP.SUPER    
  # Administrator Role
  admin: IZP.ADMIN    
- TLS, ports
- HLQs & Libraries for UMS & DB2
- dbaEncryption: "IZP.CUST.DBA.ENCRYPT"


IZP.CUST.SAMPLIB(IZPGENER) - generates IZP.CUST.JCLLIB


https://www.ibm.com/docs/en/umsfz/1.2.0?topic=ums-step-2-installing-unified-management-server

IZPA1.... N/A - allocates TEAMLIST
IZPA1V... verify  
IZPA2.... N/A - allocates USERLIST    
IZPA2V... verify    
IZPA3.... YES - allocates IZP.CUST.DBA.ENCRYPT   
IZPA3V... verify  
IZPB0R... N/A - Create a new group for surrogate users. This is not required for useSAFOnly.  
IZPB0VR.. verify  
IZPB1R... YES - Create IZP class and add to the CDT.  
IZPB1VR.. verify  
IZPB2R... YES - Add security role profiles to the IZP class.   
IZPB2VR.. verify  
IZPB3R... N/A - Create generic profiles to secure userList and teamList data sets. This is not required if useSAFOnly=true.  
IZPB3VR.. verify  
IZPB4R... YES - Create RACF IZP resource profiles to define the UMS users and their roles.  
IZPB4VR.. verify  
IZPC1R... N/A - Add surrogate users to impersonate when accessing the userList and teamList data sets during runtime. This is not required if useSAFOnly=true.  
IZPC1VR.. verify  
IZPC2R... N/A - Grant surrogate user access to the userList and teamList profiles. This is not required if useSAFOnly=true.  
IZPC2VR.. verify  
IZPD1R... YES - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS.   
IZPD1VR.. verify  
IZPD2R... N/A - Grant system programmer and started task access to PKCS #11 resources. 
 >>> IZPD2R 	Grant system programmer and started task access to PKCS #11 resources. 
 >>> This is not required if you are not using either a UMS default DBA user ID with a password (specified by the key components.izp.security.pkcs11.dbaUser) or a Db2 subsystem-specific DBA user ID with a password.
 >>> Too many double negatives. just run the fucker, 'cos it can't do any harm. 
 IZPD2VR.. verify  
 >>> //* In the output of this job, make sure that the following is true:
 >>> //*                                                                 
 >>> //* ZWEADMIN has READ access to USER.IZPTOK.                                                 
 >>> //* ZWEADMIN has CONTROL access to SO.IZPTOK.                                                   
 >>> //* ZWESLSTC has UPDATE access to USER.IZPTOK........ ( ZWESVUSR )        
 >>>                                          
IZPD3R... N/A - Create the PKCS #11 token for UMS. This is not required if you are  
 >>> Again - too many double negatives 	
 >>> Just run the fucker. 
IZPD3VR.. verify  
IZPD4R... YES - Add a new user to serve as the DBA user ID.  
IZPD4VR.. verify  
IZPD5R... YES - Connect the DBA user ID to the IZUUSER group for z/OSMF.  
IZPD5VR.. verify  
IZPD6R... YES - Grant the DBA user ID access to applications. If useSAFOnly=true, permits are not required for the surrogate users.   
IZPD6VR.. verify  
IZPD7R... YES - Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache.   
IZPD7VR.. verify  
IZPSTEPL. YES - concatenate datasets in PROCLIB member
IZPUSRMD. N/A - If useSafOnly is set to true or you are migrating from UMS 1.1, do not submit the IZPUSRMD JCL.
izp-encrypt-dba.sh
IZPIPLUG. YES - Install Zowe plugins using the zwe command.
IZPEXPIN. YES - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT
WLMPROC 

IZP.CUST.PARMLIB(IZPDB2PM)
# UMS HLQs                                                                                                   
# Sample value: HLQ.IZP.DSN                                                         
IZP_DB2_USR_HLQ: IZP.CUST                                                           
# Sample value (recommended): SIZP                                                  
IZP_DB2_USR_PREFIX: SIZP                                                            
# DB2ADM HLQs
IZP_DB2_ADB_HLQ: ADBD10                                                             
# Sample value (if using IBM Admin Tool): SADB                                      
# Sample value (otherwise, using Admin Foundation files) : SAFX                     
IZP_DB2_ADB_PREFIX: SADB                                                            


Start ZOWE 


Objects Created. ( To be deleted for a re-install )


21 'IZP.CUST.DBA.ENCRYPT'
22 'IZP.CUST.ENVIRON'    
23 'IZP.CUST.JCLLIB'     
24 'IZP.CUST.PARMLIB'    
25 'IZP.CUST.SAFXDBRM'   
26 'IZP.CUST.SAFXLLIB'   
27 'IZP.CUST.SAFXSAMP'   
28 'IZP.CUST.SAMPLIB'    


RE-DEPLOY

Delete all RACF Resources Matching IZP* 
Delete all IZP.CUST datasets, except for IZP.CUST.SAMPLIB and IZP.CUST.PARMLIB
rm -rf in /global/ums

Omit IZP.CUST.SAMPLIB(IZPALOPL) ... Allocates IZP.CUST.PARMLIB
Omit IZP.CUST.SAMPLIB(IZPCPYML) ... Copy ZWEYAML

Edit IZP.CUST.PARMLIB(ZWEYAML)
... allowselfsigned

IZP.CUST.SAMPLIB(IZPGENER) - generates IZP.CUST.JCLLIB


  
IZPA3.... YES - allocates IZP.CUST.DBA.ENCRYPT   
IZPA3V... verify  

IZPB1R... YES - Create IZP class and add to the CDT.  
IZPB1VR.. verify  

IZPB2R... YES - Add security role profiles to the IZP class.   
IZPB2VR.. verify  

>>>> ICH10321I The profile name IZP.SUPER* contains generic characters, but generics are not enabled for class IZP.  A discrete profile has been created.  
>>>> RDELETE CDT IZP               
>>>> SETROPTS RACLIST(CDT) REFRESH 
>>>> Re-RUN FROM IZPB1R

IZPB4R... YES - Create RACF IZP resource profiles to define the UMS users and their roles.  
IZPB4VR.. verify  
 
IZPD1R... YES - Define CRYPTOZ resource profiles for the PKCS #11 token for UMS.   
IZPD1VR.. verify  

IZPD2R... N/A - Grant system programmer and started task access to PKCS #11 resources. 
 >>> IZPD2R 	Grant system programmer and started task access to PKCS #11 resources. 
 >>> This is not required if you are not using either a UMS default DBA user ID with a password (specified by the key components.izp.security.pkcs11.dbaUser) or a Db2 subsystem-specific DBA user ID with a password.
 >>> Too many double negatives. just run the fucker, 'cos it can't do any harm. 
 IZPD2VR.. verify  
 >>> //* In the output of this job, make sure that the following is true:
 >>> //*                                                                 
 >>> //* ZWEADMIN has READ access to USER.IZPTOK.                                                 
 >>> //* ZWEADMIN has CONTROL access to SO.IZPTOK.                                                   
 >>> //* ZWESLSTC has UPDATE access to USER.IZPTOK........ ( ZWESVUSR )        
 >>>                                          

IZPD3R... N/A - Create the PKCS #11 token for UMS. This is not required if you are  
 >>> Again - too many double negatives 	
 >>> Just run the fucker. 
IZPD3VR.. verify  
 
IZPD4R... YES - Add a new user to serve as the DBA user ID.  
IZPD4VR.. verify  
>>> didn't bother deleting it from last time. ( IZPDBA/SYS1 )

IZPD5R... YES - Connect the DBA user ID to the IZUUSER group for z/OSMF.  
IZPD5VR.. verify  

IZPD6R... YES - Grant the DBA user ID access to applications. If useSAFOnly=true, permits are not required for the surrogate users.   
IZPD6VR.. verify  
>>> PERMIT OMVSAPPL CLASS(APPL) ID(IZPDBA)
>>> ICH06004I OMVSAPPL NOT DEFINED TO RACF

IZPD7R... YES - Creates function profiles in IZP class that are used when useSafOnly is enabled, which allow users to refresh the security cache.   
IZPD7VR.. verify  

IZPSTEPL. YES - concatenate datasets in PROCLIB member

izp-encrypt-dba.sh
/usr/lpp/IBM/izp/v1r2m0/bin/ums/opt/bin 


IZPIPLUG. YES - Install Zowe plugins using the zwe command.

IZPEXPIN. YES - LAUNCH THE IZP EXPERIENCE INTEGRATION SCRIPT



```
