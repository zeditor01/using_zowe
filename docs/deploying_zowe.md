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
* The base applications provided by ZOWE all depend on calling services provided by z/OSMF.
* zowe uses a certificate (optionally self-signed) for authentication and encryption purposes.
* A RACF keyring stores that certificate, together wth the CA Certificate used by z/OSMF.
* ZOWE listens to browser clients on port 7554 by default.
* The entire configuration of zowe is stored in a YAML file, which is central to installation, configuration and runtime.

![zowe_deploy02](/images/zowe_deploy02.JPG)

This worked example is **NOT** a replacement for the [ZOWE documentation](https://docs.zowe.org/stable/user-guide/install-overview).
It is a worked example that is intened to show readers a simple overview of deploying ZOWE. 
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
* 3 Methods of accessing the ZOWE code
* Installing with the COnvenience Build Method
* The zowe.yaml file, which the central configuration point for ZOWE
* Executing the script to install the instance
* Executing the script to configure the instance
* Operating and using ZOWE

## 2. Installing the ZOWE product code.

Creating and Mounting a ZFS

Installing / Unpacking the ZOWE product COde

## 3. Editing the zowe.yaml file.

Section 1

Section 2

Section N

## 4. Installing the ZOWE instance libraries.

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
