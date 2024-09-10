![zowelogo](/images/zowelogo.JPG)

# Using ZOWE
Worked Examples of Deploying ZOWE and Associated IBM Db2 Tools.

## Imagine...
* A productive browser-based graphical user interface to perform a wide range of Db2 z/OS Database Administration work tasks. 
* A Catalog of APIs to provide callable database services that can be re-used by multiple integrated tools.
* All provided from services running in z/OS, without needing to deploy any clients or servers outside of z/OS.

## ZOWE offers:
Zowe offers modern interfaces to interact with z/OS similar to what you may experience on cloud platforms today. 

The home of ZOWE is at [www.zowe.org](https://www.zowe.org/)

## IBM offers:
A growing range of zowe-based tools for Db2 z/OS, based on the IBM Unified Management Server for z/OS, and the IBM Db2 Administration Foundation.

## This github repository offers:
A structured set of worked examples for using the zowe-based DB2 tooling, at the pages linked below....

Deployment Worked Examples
* [Deploying ZOWE](https://github.com/zeditor01/using_zowe/blob/main/docs/deploying_zowe.md)   
* [Deploying Unified Management Server for z/OS and Db2 Administration Foundation](https://github.com/zeditor01/using_zowe/blob/main/docs/deloying_ums_and_db2adminfoundation.md)
* [Configuring SQL Tuning Services within Db2 Administration Foundation](https://github.com/zeditor01/using_zowe/blob/main/docs/configure_sql_tuning.md) Not yet written    
* [Deploying Db2 Automation Experience](https://github.com/zeditor01/using_zowe/blob/main/docs/deploying_db2automationexperience.md) Not yet written
* [Deploying Db2 Devops Experience](https://github.com/zeditor01/using_zowe/blob/main/docs/deploying_db2devopsexperience.md) Not yet written

Usage Worked Examples
* [Using Db2 Administration Foundation](https://github.com/zeditor01/using_zowe/blob/main/docs/using_db2adminfoundation.md) Not yet written
* [Using Db2 Automation Experience](https://github.com/zeditor01/using_zowe/blob/main/docs/using_db2automationexperience.md) Not yet written
* [Using Db2 Devops Experience](https://github.com/zeditor01/using_zowe/blob/main/docs/using_db2evopsexperience.md) Not yet written

# The Sequence of Steps described in this Repository

This github repository is an audit trail of the following journey:

## Starting Point : z/OS system with z/OSMF
The starting point on your ZOWE journey is a fully functional z/OSMF service. Most sites should have this up and running already, because it is the recommended vehicle for installing software from ShopZ.

z/OSMF supports secure connections from a web browser, to allow the user to invoke a range of applications and services. The CA Certificate of z/OSMF must be imported into the Web Browsers trust store to enable secure connections.

![zowe_deploy01](/images/zowe_deploy01.JPG)

## Step 1 : Deploy ZOWE
ZOWE uses z/OSMF services. It's installation service will create a new Keyring and certificates to allow secure commincations from browsers, and secure communications to consume z/OSMF services.

![zowe_deploy02](/images/zowe_deploy02.JPG)

## Step 2 : Deploy UMS
UMS depends upon ZOWE as a pre-requisite. The Unified Management Service App runs in ZOWE and invokes services that are provided by Unified Management Server.

UMS utilised the same keyring and certificates as ZOWE. It is secured using the RACF IZP class. It is strongly recommended to configure UMS to execute in SAFOnly mode, so that all access is controlled by the SAF (RACF in this worked example). 

UMS is installed into ZOWE, so that when ZOWE starts, it automatically starts UMS.

![zowe_deploy02](/images/zowe_deploy03.JPG)

## Step 3 : Deploy Db2 Administration Foundation
Db2 Administration Foundation is a set of base services for Db2 tools running under ZOWE. It's services include
* Catalog Navigation
* Command Execution
* SQL Exection
* SQL Tuning
* DDL Generation

These base services are provided at no charge. Additional Experiences ($ licensed) will sit on top of Db2 Administration Foundation services, and add new capabilities to the stack.

![zowe_deploy04](/images/zowe_deploy04.JPG)

## Step 4 : Deploy additional Db2 (and IMS) experiences

Adding additional experiences to the stack will be done in a very similar way to deploying Db2 Administration Foundation.

![zowe_deploy05](/images/zowe_deploy05.JPG)


