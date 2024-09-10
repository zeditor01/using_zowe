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

ZOWE depends upon z/OSMF, RACF and TCPIP infrastructure, as illustrated below.
![zowe_deploy02](/images/zowe_deploy02.JPG)

ZOWE documentation

ZOWE architecture

Certificates

This Worked Example

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
