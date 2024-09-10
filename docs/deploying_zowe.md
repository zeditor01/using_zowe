[Back to Index](https://github.com/zeditor01/using_zowe/blob/main/README.md)

Last Update to this page: 9 September 2024

# Deploying ZOWE

***Purpose of this Chapter***
ZOWE is the foundation for the new generation of Db2 tools. The scope of this chapter is limited to a worked example of installing and deploying ZOWE.

## Contents
1. Planning for ZOWE.
2. Installing the ZOWE product code.
3. Editing the zowe.yaml file.
4. Installing the ZOWE instance libraries.
5. Configuring the ZOWE instance.
6. Operating ZOWE.
7. Using the ZOWE Base Apps.

## 1. Planning for ZOWE.

### 1.1 How ZOWE fits in z/OS.
ZOWE depends upon z/OSMF, RACF and TCPIP infrastructure. 
* The base applications provided by ZOWE all depend on calling services provided by z/OSMF. (Blue boxes below)
* ZOWE uses a certificate (optionally self-signed) for authentication and encryption purposes.
* A RACF keyring stores that certificate, together wth the CA Certificate used by z/OSMF, so that it can call z/OSMF services.
* ZOWE listens to browser clients on port 7554 (default port, can be changed).
* The entire configuration of zowe is stored in a YAML file, which is central to installation, configuration and runtime.

![zowe_deploy02](/images/zowe_deploy02.JPG)

This worked example is **NOT** a replacement for the [ZOWE installation documentation](https://docs.zowe.org/stable/user-guide/install-overview).
It is a worked example that is intened to show readers a chronological view of steps to perform. 
Hopefully it will also make it easier to consume the ZOWE documentation.

### 1.2 ZOWE architecture
In this worked example, we are only deploying the ZOWE Application Framework, running on z/OS. That is the only component that we need to invoke the ZOWE-based Db2 Tools.

ZOWE has other components, which you can ( and should) learn about [here](https://docs.zowe.org/stable/getting-started/overview).
1. Zowe Launcher
2. API Mediation Layer
3. Zowe Application Framework
4. Zowe CLI
5. Zowe Explorer
6. Zowe Client Software Development Kits SDKs

### 1.3 Scope of Activities in this worked example
In this worked example, we will cover
* The pre-requisites before installing ZOWE
* 4 Methods of accessing the ZOWE code
* Installing with the Convenience Build Method
* The zowe.yaml file, which the central configuration point for ZOWE
* Executing the script to install the instance
* Executing the script to configure the instance
* Operating and using ZOWE

### 1.4 Pre-Requisite Software
You must ensure that some prequisite software ( z/OSMF, Java, NodeJS etc...) is installed on z/OS before you install zowe. The main System Requirements are listed here
* AXR (System REXX)
* CEA (Common Event Adapter - of z/OSMF)
* CIM (Common Information Model - of z/OSMF)
* CONSOLE and CONSPROF commands must exist in the authorised command table
* Java (V8 or later)
* NodeJS
* TSO Region Size - minimum 65,536 KB
* Userids - OMVS Segment

Please refer to the current list of pre-requisites [here](https://docs.zowe.org/stable/user-guide/zosmf-install)  before you start.

I am using the May 2024 ADCD build of z/OS (which is based on z/OS V3.1). It included all the pre-requisites, but CFZCIM was not started by default. I changed USER.Z31A.PARMLIB(VTAMALL) to include the ```S CFZCIM``` command.

## 2. Installing the ZOWE product code.

There are 4 options for installing the Zowe server-side component
1. Convenience build (unpack a pax file)
2. SMPE build (FMID + PTFs)
3. Portable Software Instance
4. Containerised build

You can download the server-side component from the Zowe website, or you can order a PSI from ShopZ. I chose the the convenience build. I download V2.17 for this worked example.


With the zowe-convenience build, I downloaded ```zowe-2.17.0.pax``` and transferred it to my smp downloads folder at ```/u/smpe/smpnts/zcb```.

I then allocated a ZFS and mounted it at ```/usr/lpp/zwe/zwe217```. Permanent PARMLIB mount specification below.

```
MOUNT FILESYSTEM('ZOWE217.ZFS')       
      TYPE(ZFS)                       
      MODE(RDWR)                      
      NOAUTOMOVE                      
      MOUNTPOINT('/usr/lpp/zwe/zwe217') 
```

I then copied the pax file from ```/u/smpe/smpnts/zcb``` zcb into target ZFS ```/usr/lpp/zwe/zwe217``` for deployment.

Unpack the pax file with the Command : ```pax -rvf zowe-2.17.0.pax```

Results in
```
IBMUSER:/Z31A/usr/lpp/zwe/zwe217: >ls -al
total 1039280
drwxr-xr-x   8 OMVSKERN SYS1        8192 Jul 19 23:18 .
drwxr-xr-x   5 OMVSKERN OMVSGRP     8192 Jul 19 22:43 ..
-rw-r--r--   1 OMVSKERN SYS1        2350 Jul 18 08:31 DEVELOPERS.md
-rw-r--r--   1 OMVSKERN SYS1        3772 Jul 18 08:31 README.md
drwxr-xr-x   5 OMVSKERN SYS1        8192 Jul 18 08:31 bin
drwxr-xr-x  19 OMVSKERN SYS1        8192 Jul 18 08:34 components
-rw-r--r--   1 OMVSKERN SYS1       25982 Jul 18 08:31 example-zowe.yaml
drwxr-xr-x   7 OMVSKERN SYS1        8192 Jul 18 08:34 files
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:35 fingerprint
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:31 licenses
-rw-r--r--   1 OMVSKERN SYS1       14213 Jul 18 08:31 manifest.json
drwxr-xr-x   2 OMVSKERN SYS1        8192 Jul 18 08:31 schemas
-rw-r-----   1 OMVSKERN SYS1     531675648 Jul 19 22:45 zowe-2.17.0.pax
```

This matches the file structure described by the Zowe documentation

![zowe_directory](/images/zowedir.JPG)


## 3. Editing the zowe.yaml file.

Within /usr/lpp/zwe/zwe217 , copy example-zowe.yaml to zowe.yaml for editing.

The entire Zowe deployment is defined by the zowe.yaml file. This configuration file is used by
1. the ZWE INSTALL script to create the ZOWE installation datasets
2. the ZWE INIT script to customise the ZOWE instance
3. at Zowe runtime

If you've not come across yaml files yet, yaml stands for 'yet another markup language'. The zowe.yaml file is documented [here](https://docs.zowe.org/stable/appendix/zowe-yaml-configuration/). It has 6 sections

1. zowe (Defines global configurations specific to Zowe, including default values)
2. java (Defines Java configurations used by Zowe components)
3. node (Defines node.js configurations used by Zowe components)
4. zOSMF (Tells Zowe your z/OSMF configurations)
5. components (Defines detailed configurations for each Zowe component or extension)
6. haInstances (Defines customized configurations for each High Availability (HA) instance)

I prepared the following [zowe.yaml](https://github.com/zeditor01/zowetools/blob/main/code/zowe.yaml) file for my system. ***Hint: right mouse click + open link in new tab.***

Note the following parameters

setup-dataset section
* line 43 provides the HLQ where the ZOWE datasets will be installed later on using the 'ZWE INSTALL' script
* line 46 specifies your chosen proclib where the startup procedures will be created
* line 49 specifies your chose parmlib for plugins
* line 56 specifies the PDS to create the customisation JCL members in
* line 58 specifies the loadlib from the chosen installation method (convenience build in my case)
* line 60 specifies the APF-authorised loadlib from the chosen installation method (convenience build in my case)
* line 63 specifies the APF authorized LOADLIB for Zowe ZIS Plugins from the chosen installation method (convenience build in my case)

setup-security section (needs to be filled in to use RACF and RACF keyrings)
* line 69 specifies that we will use RACF security. This is an opensource product, and prepares security configurations for multiple security products)
* line 71 - 77 specifies the RACF groups to create and use (I used ZWEADMIN for all three groups)
* lines 81 and 83 specifies the user for the main Zowe task (ZWESVUSR) and for the ZIS server (ZWESIUSR)
* lines 85 - 91 specifies the started task names ZWESLSTC (main server) ZWESISTC (zis server) and ZWESASTC (zis auxilliary server)

certificate section. The example zowe.yaml file provides templates for 5 different certificate configurations. I chose scenario 3 - Zowe generated z/OS Keyring with Zowe generated certificates.
* line 180 specifies that I wish to use a RACK keyring to hold certificates
* line 185 specifies the Keyring name (ZoweKeyring)
* line 188 specifies the label of my certificate
* line 190 specifies the label of my CA certificate
* line 193 - 200 specifies options distinguished name values
* line 202 ensures that the certificate won't expire until well after i retire
* line 208 and 210 specify the hostname (s0w1.dal-ebis.ihost.com) and IP address (192.168.1.171) for my z/OS system

cacheing service section
line 259 - 256 specifies to use Non-RLS VSAM for the cacheing service

runtime directories
* line 282 specifies the runtimeDirectory: "/usr/lpp/zwe/zwe217"
* line 286 specifies the logDirectory: /global/zowe/logs
* line 290 specifies the workspaceDirectory: /global/zowe/workspace
* line 294 specifies the extensions directory, where plugins will be linked: /global/zowe/extensions

ports
* line 364 specifies the port that Zowe listens for browser connections on

generated certificates: Leave this section blank. It will be updated during the 'ZWE INIT' Script
* line 396 - 420 specify the RACF keystore and truststore to be used for certificates

java
* line 447 specifies the path to Java 8. Note that Zowe V2.18 depends on Java 8 for the install, but can run with current Java 17.

nodeJS
* line 462 specifies the path to nodeJS

z/OSMF
* lines 471 - 477 specify the connection details for z/OSMF
  
I left everything else from the example zowe.yaml file to default.

Be careful of case sensitive parameters (paths and certificate details)

Once you've edited the zowe.yaml file, everything else should flow easily.


## 4. Installing the ZOWE instance libraries.


### 4.1 environment variables
export NODE_HOME=/usr/lpp/IBM/cnj/v20r11
export PATH=/usr/lpp/IBM/cnj/v20r11/bin:$PATH
export PATH=/usr/lpp/zwe/zwe217/bin:$PATH


Invoking the Installer

## 5. Configuring the ZOWE instance.

Invoking the Configuration

## 6. Operating ZOWE.

PARMLIB

Automated Operations

## 7. Using the ZOWE Base Apps.

JES Explorer

Editor

TN3270

## 8. Subsequent Upgrades.

Parallel ZOWE instances

Upgrading a single ZOWE instance


```
ZOWE

unpack the convenience install tarball

Edit /usr/lpp/zwe/zwe217/zowe.yaml

zwe install -v -c /usr/lpp/zwe/zwe217/zowe.yaml

zwe init --update-config --allow-overwrite -v -c /usr/lpp/zwe/zwe217/zowe.yaml

Check certificates
racdcert id(ZWESVUSR) listring(ZoweKeyring)                           
  Ring:                                                               
  >ZoweKeyring<                                                  
  Certificate Label Name             Cert Owner     USAGE      DEFAULT
  --------------------------------   ------------   --------   -------
  zowes0w1ca                         CERTAUTH       CERTAUTH     NO   
  zowes0w1                           ID(ZWESVUSR)   PERSONAL     YES  
  zOSMFCA                            CERTAUTH       CERTAUTH     NO   
  
zwe config validate -c /usr/lpp/zwe/zowe/zowe.yaml

USER.Z31A.PARMLIB(SCHEDAL)
PPT PGMNAME(ZWESIS01) NOSWAP KEY(4) /* Zowe cross memory           */
PPT PGMNAME(ZWESAUX)  NOSWAP KEY(4) /* Zowe auxiliary (AUX)        */

S ZWESISTC,REUSASID=YES
S ZWESLSTC  

https://s0w1.dal-ebis.ihost.com:7554/zlux/ui/v1

```
