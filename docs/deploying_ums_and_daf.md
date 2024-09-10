[Back to Index](https://github.com/zeditor01/using_zowe/blob/main/README.md)

Last Update to this page: 9 September 2024

# Deploying Unified Managament Server and Db2 Administration Foundation.

***Purpose of this Chapter***
The scope of this chapter is a worked example of deploying UMS and Db2 Administration Foundation. This is really the entry level scope of installation effort to get started with Db2 Tools for ZOWE.

## Contents
1. Planning for UMS and DAF.
2. Installing the UMS and DAF product code.
3. Editing the ZWEYAML parmlib file.
4. Installing the UMS Instance.
5. Configuring the UMS instance.
6. Operating UMS.
7. Using the Unified Server Application with Db2 Administration Foundation.
8. Subsequent Upgrades.


## 1. Planning for UMS and DAF.

### 1.1 How UMS fits in z/OS.
UMS depends upon ZOWE as a pre-requisite. The Unified Management Service App runs in ZOWE and invokes services that are provided by Unified Management Server.

UMS utilised the same keyring and certificates as ZOWE. It is secured using the RACF IZP class. It is strongly recommended to configure UMS to execute in SAFOnly mode, so that all access is controlled by the SAF (RACF in this worked example). 

UMS is installed into ZOWE, so that when ZOWE starts, it automatically starts UMS.

DAF is simply an extra set of services called by UMS to provide the basic functions for Db2 Catalog Navigation, Command Execution, SQL Execution and SQL Statement Tuning.

![zowe_deploy04](/images/zowe_deploy04.JPG)

### 1.2 UMS and DAF documentation
This worked example is **NOT** a replacement for the [UMS installation documentation](https://www.ibm.com/docs/en/umsfz/1.2.0).
It is a worked example that is intened to show readers a chronological view of steps to perform. 
Hopefully it will also make it easier to consume the ZOWE documentation.


### 1.3 Pre-Requisite Software
The hardware and software pre-requisites for UMS and the Db2 experiences are documented [here](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-prerequisite-hardware-software)

In addition to z/OS V2.4 or later, ICSF and RACF, minimum versions of ZOWE, and a number of PTF levels are documented at the link above. You should check the pre-requisites before proceeding.

### 1.4 Before you Begin
The Knowledge Center contains a ["Before You Begin"](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-before-you-begin) section that is very important. 
If it seems challenging at first sight, don't worry. I found it very difficult to consume until **after** I had completed the deployment. 
Follow this worked example to get a better understanding of the deploymemt process. Then return to this sections before you start planning for your own deployment.

### 1.5 Authentication Strategy
Older versions of UMS supported a dataset-based authentication system. This is being depracated in favour of SAF only authentication. The V1.2 knowledge centre still contains details of both authentication models. 
You need to read the documentation carefully to ensure that you implement SAFOnly security. A summary of the change can be found [here](https://community.ibm.com/community/user/datamanagement/blogs/jrn-thyssen1/2023/08/30/security-enhancement-for-ibm-unified-management-se)

## 2. Installing the ZOWE product code.

UMS V1.2 and DAF V1.2 should both be ordered as a portable software instance from ShopZ, and installed using the z/OSMF software management tool. Executing a PSI installation is standard fare for any systems programmer, and not repeated here.

### 2.1 z/OS datasets installed for UMS
The PSI installation is outside the scope of this document. I performed a standard SMPE PSI installation to HLQ "IZP", resulting in the following z/OS Datasets
```
Data Set Names / Objects                       Volume
---------------------------------------------- ------
'IZP.AIZPBASE'                                 A3USR8
'IZP.AIZPBIN'                                  A3USR3
'IZP.AIZPCUSA'                                 A3USR4
'IZP.AIZPCUSC'                                 A3USR3
'IZP.AIZPCUSR'                                 A3USR4
'IZP.AIZPCUST'                                 A3USR8
'IZP.AIZPLOAD'                                 A3USR3
'IZP.AIZPPARM'                                 A3USR5
'IZP.AIZPPAX'                                  A3USR2
'IZP.AIZPREXX'                                 A3USR2
'IZP.AIZPSAMP'                                 A3USR4
'IZP.CPAC.DB2.PARMLIB'                         A3USR4
'IZP.CPAC.PDFPD'                               A3USR7
'IZP.CPAC.PROD.PDF'                            A3USR2
'IZP.CPAC.PROPVAR'                             A3USR2
'IZP.CPAC.SCPPCENU'                            A3USR7
'IZP.CPAC.SCPPLOAD'                            A3USR6
'IZP.CPAC.WORKFLOW'                            A3USR6
'IZP.CPACDB2.DOCLIB'                           A3USR2
'IZP.CPACDB2.SAMPLIB'                          A3USR6
'IZP.DBA.ENCRYPT'                              A3USR2
'IZP.ENVIRON'                                  A3USR5
'IZP.JCLLIB'                                   A3USR3
'IZP.OMVS.SIZPROOT'                                  
'IZP.OMVS.SIZPROOT.DATA'                       A3USR5
'IZP.PARMLIB'                                  A3USR2
'IZP.SAFXDBRM'                                 A3USR2
'IZP.SAFXLLIB'                                 A3USR6
'IZP.SAFXSAMP'                                 A3USR7
'IZP.SAMPLIB'                                  A3USR3
'IZP.SIZPBASE'                                 A3USR2
'IZP.SIZPCUSA'                                 A3USR7
'IZP.SIZPCUSC'                                 A3USR3
'IZP.SIZPCUSR'                                 A3USR4
'IZP.SIZPCUST'                                 A3USR6
'IZP.SIZPLOAD'                                 A3USR4
'IZP.SIZPPARM'                                 A3USR3
'IZP.SIZPREXX'                                 A3USR8
'IZP.SIZPSAMP'                                 A3USR3
'IZP.SMPE.DB2.DLIB.CSI'                              
'IZP.SMPE.DB2.DLIB.CSI.DATA'                   A3USR6
'IZP.SMPE.DB2.DLIB.CSI.INDEX'                  A3USR6
'IZP.SMPE.DB2.GLOBAL.CSI'                            
'IZP.SMPE.DB2.GLOBAL.CSI.DATA'                 A3USR6
'IZP.SMPE.DB2.GLOBAL.CSI.INDEX'                A3USR6
'IZP.SMPE.DB2.SMPGLOG'                         A3USR4
'IZP.SMPE.DB2.SMPGLOGA'                        A3USR8
'IZP.SMPE.DB2.SMPPTS'                          A3USR4
'IZP.SMPE.DB2.TARGET.CSI'                            
'IZP.SMPE.DB2.TARGET.CSI.DATA'                 A3USR2
'IZP.SMPE.DB2.TARGET.CSI.INDEX'                A3USR2
'IZP.SMPE.DB2D300.SMPDLOG'                     A3USR3
'IZP.SMPE.DB2D300.SMPDLOGA'                    A3USR5
'IZP.SMPE.DB2T300.SMPMTS'                      A3USR7
'IZP.SMPE.DB2T300.SMPSCDS'                     A3USR2
'IZP.SMPE.DB2T300.SMPSTS'                      A3USR6
'IZP.SMPE.DB2T300.SMPTLOG'                     A3USR3
'IZP.SMPE.DB2T300.SMPTLOGA'                    A3USR4
'IZP.TEAMLIST'                                 A3USR8
'IZP.USERLIST'                                 A3USR7
'IZP.USERLIST'                                 A3USR7 
```

### 2.2 USS filesystem mounted for UMS
There is a ZFS filesystem that I mounted permenantly in PARMLIB as follows
```                               
MOUNT FILESYSTEM('IZP.OMVS.SIZPROOT')            
      TYPE(ZFS)                                  
      MODE(RDWR)                                 
      NOAUTOMOVE                                 
      MOUNTPOINT('/usr/lpp/IBM/izp/v1r2m0/bin')  
```

And the contents of the ZFS are as follows
```
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >ls -al
total 274016
drwxr-xr-x  17 OMVSKERN SYS1        8192 Jul  4 22:23 .
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Jul  4 22:33 ..
drwxr-xr-x   2 OMVSKERN OMVSGRP     8192 Jun 13 05:25 IBM
-rwxrwxr-x   2 OMVSKERN OMVSGRP     1425 Jun 13 05:25 IZPCP
-rwxrwxr-x   2 OMVSKERN OMVSGRP     7035 Jun 13 05:25 IZPUNPAX
-rwxrwxr-x   2 OMVSKERN OMVSGRP     6088 Jun 13 05:25 IZPUNPX2
drwxrwxr-x   3 OMVSKERN OMVSGRP     8192 Nov 28  2022 cidb
-rwxrwxr-x   2 OMVSKERN OMVSGRP   483840 Jun 13 05:25 cidb.pax
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Dec 15  2023 db2-auth
-rwxrwxr-x   2 OMVSKERN OMVSGRP    32320 Jun 13 05:25 db2-auth.pax
drwxr-xr-x   5 OMVSKERN OMVSGRP     8192 Dec 18  2023 ivp
-rwxrwxr-x   2 OMVSKERN OMVSGRP   516160 Jun 13 05:25 ivp.pax
-rwxrwxr-x   2 OMVSKERN OMVSGRP  89349120 Jun 13 05:25 izpserve.pax
-rwxrwxr-x   2 OMVSKERN OMVSGRP     1153 Jun 13 05:25 manifest.yaml
drwxr-xr-x   2 OMVSKERN OMVSGRP     8192 Dec  1  2022 openssl
-rwxrwxr-x   2 OMVSKERN OMVSGRP  26417680 Jun 13 05:25 openssl.pax
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Nov 28  2022 platform
-rwxrwxr-x   2 OMVSKERN OMVSGRP    32320 Jun 13 05:25 platform-activation.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Dec 18  2023 ums
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Jun 21  2023 ums-security
-rwxrwxr-x   2 OMVSKERN OMVSGRP   483840 Jun 13 05:25 ums-security.pax
drwxr-xr-x   7 OMVSKERN OMVSGRP     8192 Dec 15  2023 unified-ui
-rwxrwxr-x   2 OMVSKERN OMVSGRP  17224720 Jun 13 05:25 unified-ui.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Sep 21  2023 zos-newton-daj
-rwxrwxr-x   2 OMVSKERN OMVSGRP   838720 Jun 13 05:25 zos-newton-daj.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-db2ifi
-rwxrwxr-x   2 OMVSKERN OMVSGRP   612880 Jun 13 05:25 zos-newton-db2ifi.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Aug  4  2023 zos-newton-discovery
-rwxrwxr-x   2 OMVSKERN OMVSGRP  1548320 Jun 13 05:25 zos-newton-discovery.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-registry
-rwxrwxr-x   2 OMVSKERN OMVSGRP   548400 Jun 13 05:25 zos-newton-registry.pax
drwxrwxr-x   4 OMVSKERN OMVSGRP     8192 Nov 28  2022 zos-newton-security
-rwxrwxr-x   2 OMVSKERN OMVSGRP   612880 Jun 13 05:25 zos-newton-security.pax
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Sep 21  2023 zss-data-provider
-rwxrwxr-x   2 OMVSKERN OMVSGRP  1193520 Jun 13 05:25 zss-data-provider.pax
```

### 2.3 z/OS datasets installed for UMS
The PSI installation is outside the scope of this document. I performed a standard SMPE PSI installation to HLQ "AFX", resulting in the following z/OS Datasets
```
'AFX.AAFXBASE'                                 A3USR2
'AFX.AAFXBIN'                                  A3USR7
'AFX.AAFXSAMP'                                 A3USR3
'AFX.CPAC.DB2.PARMLIB'                         A3USR6
'AFX.CPAC.PDFPD'                               A3USR8
'AFX.CPAC.PROD.PDF'                            A3USR6
'AFX.CPAC.PROPVAR'                             A3USR5
'AFX.CPAC.SCPPCENU'                            A3USR6
'AFX.CPAC.SCPPLOAD'                            A3USR4
'AFX.CPAC.WORKFLOW'                            A3USR8
'AFX.CPACDB2.DOCLIB'                           A3USR2
'AFX.CPACDB2.SAMPLIB'                          A3USR5
'AFX.OMVS.SAFXROOT'                                  
'AFX.OMVS.SAFXROOT.DATA'                       A3USR8
'AFX.SAFXBASE'                                 A3USR8
'AFX.SAFXSAMP'                                 A3USR2
'AFX.SMPE.DB2.DLIB.CSI'                              
'AFX.SMPE.DB2.DLIB.CSI.DATA'                   A3USR5
'AFX.SMPE.DB2.DLIB.CSI.INDEX'                  A3USR5
'AFX.SMPE.DB2.GLOBAL.CSI'                            
'AFX.SMPE.DB2.GLOBAL.CSI.DATA'                 A3USR5
'AFX.SMPE.DB2.GLOBAL.CSI.INDEX'                A3USR5
'AFX.SMPE.DB2.SMPGLOG'                         A3USR6
'AFX.SMPE.DB2.SMPGLOGA'                        A3USR5
'AFX.SMPE.DB2.SMPPTS'                          A3USR8
'AFX.SMPE.DB2.TARGET.CSI'                            
'AFX.SMPE.DB2.TARGET.CSI.DATA'                 A3USR7
'AFX.SMPE.DB2.TARGET.CSI.INDEX'                A3USR7
'AFX.SMPE.DB2D300.SMPDLOG'                     A3USR8
'AFX.SMPE.DB2D300.SMPDLOGA'                    A3USR3
'AFX.SMPE.DB2T300.SMPMTS'                      A3USR8
'AFX.SMPE.DB2T300.SMPSCDS'                     A3USR5
'AFX.SMPE.DB2T300.SMPSTS'                      A3USR6
'AFX.SMPE.DB2T300.SMPTLOG'                     A3USR7
'AFX.SMPE.DB2T300.SMPTLOGA'                    A3USR8
```

### 2.2 USS filesystem mounted for UMS
There is a ZFS filesystem that I mounted permenantly in PARMLIB as follows
```                                                      
 MOUNT FILESYSTEM('AFX.OMVS.SAFXROOT')          
       TYPE(ZFS)                                
       MODE(RDWR)                               
       NOAUTOMOVE                               
       MOUNTPOINT('/usr/lpp/IBM/afx/v1r2m0/bin')
```

And the contents of the ZFS are as follows
```
IBMUSER:/Z31A/usr/lpp/IBM/afx/v1r2m0/bin: >ls -al
total 18928
drwxr-xr-x   5 OMVSKERN SYS1        8192 Jul 19 20:04 .
drwxr-xr-x   3 OMVSKERN OMVSGRP     8192 Jul 19 20:10 ..
-rwxrwxr-x   2 OMVSKERN OMVSGRP     6079 Jul 18 20:32 AFXUNPAX
-rwxrwxr-x   2 OMVSKERN OMVSGRP     6088 Jul 18 20:32 AFXUNPX2
drwxr-xr-x   2 OMVSKERN OMVSGRP     8192 Jul 18 20:32 IBM
drwxr-xr-x   4 OMVSKERN OMVSGRP     8192 Dec 15  2023 admin-fdn-db2
-rwxrwxr-x   2 OMVSKERN OMVSGRP    64560 Jul 18 20:32 db2-admin-fdn-opt.pax
-rwxrwxr-x   2 OMVSKERN OMVSGRP   161280 Jul 18 20:32 db2-admin-fdn-var.pax
drwxr-xr-x   5 OMVSKERN OMVSGRP     8192 Dec 15  2023 single_ddl
-rwxrwxr-x   2 OMVSKERN OMVSGRP  9386560 Jul 18 20:32 single_ddl.pax
```

### 2.5 Program Control Authorisation

UMS Zowe plug-ins require Program Control authorization. In order to tag the files with this bit, the SMP/E install user requires BPX.FILEATTR.PROGCTL permission on the system.

```
extattr +p */zssServer/lib/*
```

So, I opened an ssh terminal, and navigated to the ```components.izp.runtimeDirectory``` directory, and executed the commad

```
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >pwd
/Z31A/usr/lpp/IBM/izp/v1r2m0/bin
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >extattr +p */zssServer/lib/*
IBMUSER:/Z31A/usr/lpp/IBM/izp/v1r2m0/bin: >
```


## 3. Editing the ZWEYAML parmlib file

As with ZOWE, the entire installation and configuration of UMS is controlled by a single YAML file. If you choose wisely at this point, the installation should flow smoothly.

Editing the ZWEYAML file is performed as one step of the UMS installation in the next section. We will review the ZWEYAML file ahead of time, so that when we start the installation tasks we just follow a set of simple tasks.

The actual parameters files that I used in this example and included within this github repository, links below. In addition to the ZWEYAML file

[ZWEYAML here](https://github.com/zeditor01/using_zowe/blob/main/samples/ZWEYAML.TXT)

[IZPDB2PM here](https://github.com/zeditor01/using_zowe/blob/main/samples/IZPDB2PM.TXT)

[IZPDAFPM here](https://github.com/zeditor01/using_zowe/blob/main/samples/IZPDAFPM.TXT)


Write Detailed notes on the reasons behind each of these choices

* include DAF so there is something to test with UMS
* useSAFOnly to fit with the strategic authentication method that will continue to be supported
* credential management tokens - MFA ?
* PASSWORD is the only option for DB2
* profileQualifier has loads of ramifications for the IZP RACF Class & Generic Profiles
* DBAUSER - create one & use encrypted PWD as a RACF token
* Point to ZOWE keyring & certificates
* etc etc

```
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
```

## 4. Installing the UMS instance.

The tasks below are my selection of tasks from [this](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=ums-step-2-installing-unified-management-server)

```
Stop ZOWE
Stop ZOWE Cross Memory Server
    
Copy SIZPSAMP to another data set of the same attributes IZP.CUST.SAMPLIB ( members  IZPALOPL,  IZPCPYML, IZPCPYM2, IZPGENER, IZPMIGRA )

Edit and submit IZP.CUST.SAMPLIB(IZPALOPL) ... Allocates IZP.CUST.PARMLIB
   
Edit and submit IZP.CUST.SAMPLIB(IZPCPYML) ... Creates the ZWEYAML default PARMLIB member (to be edited).

(Migration Only) IZPMIGRA Migrates current values from version 1.1 to version 1.2.

Edit IZP.CUST.PARMLIB(ZWEYAML) 

IZP.CUST.SAMPLIB(IZPGENER) - generates IZP.CUST.JCLLIB
```

## 5. Configuring the UMS instance.

continuation of [this](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=ums-step-2-installing-unified-management-server)

Work your way thru the JCLs
```
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
```

## 6. Operating UMS.

Just Start ZOWE same as before

Watch Startup

Sample joblog [here](https://github.com/zeditor01/using_zowe/blob/main/samples/ums_stc_out.txt)


## 7. Using the Unified Management Experience App

Include Db2 subsystem register steps

UMS01

![ums01](/images/ums01.JPG)

UMS02

![ums02](/images/ums02.JPG)

UMS03

![ums03](/images/ums03.JPG)

UMS04

![ums04](/images/ums04.JPG)

UMS05

![ums05](/images/ums05.JPG)

UMS06

![ums06](/images/ums06.JPG)

UMS07

![ums07](/images/ums07.JPG)

UMS08

![ums08](/images/ums08.JPG)

UMS09

![ums09](/images/ums09.JPG)

UMS10

![ums10](/images/ums10.JPG)


UMS11

![ums11](/images/ums11.JPG)

UMS12

![ums12](/images/ums12.JPG)

UMS13

![ums13](/images/ums13.JPG)

UMS14

![ums14](/images/ums14.JPG)

UMS15

![ums15](/images/ums15.JPG)

UMS16

![ums16](/images/ums16.JPG)

UMS17

![ums17](/images/ums17.JPG)

UMS18

![ums18](/images/ums18.JPG)

UMS19

![ums19](/images/ums19.JPG)



## 8. Subsequent Upgrades.

[PTFS](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-installing-program-temporary-fix-ptf)

[POST ZOWE Upgrades](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-post-zowe-upgrade-tasks-ums)

[Post DB2 Upgrade](https://www.ibm.com/docs/en/umsfz/1.2.0?topic=installation-configuring-ums-after-upgrading-db2-zos)



### Scratch Pad Area

```


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
