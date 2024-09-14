![zowelogo](/images/zowelogo.JPG)


# Revolutionizing Mainframe Interactions with ZOWE

Imagine a world where mainframe operations are as intuitive and accessible as modern cloud platforms. This is the reality ZOWE brings to z/OS environments, offering a suite of open-source tools that transform how developers and system administrators interact with mainframe systems.

ZOWE addresses the needs of many IT groups, such as application developers, systems programmers, devops engineers and operations support staff. It also supports Database Engineering specialists.

Streamlined Db2 z/OS Database Administration
* ZOWE provides a powerful, browser-based graphical user interface that simplifies a wide range of Db2 z/OS Database Administration tasks. 
* Execute complex queries with ease, and invoke SQL tuning services
* Manage database objects efficiently
* Troubleshoot issues quickly

API-Driven Integration with a comprehensive API catalog, offering:
* Reusable database services
* Seamless integration with existing tools
* Enhanced interoperability between mainframe and distributed systems
* Simplified development of new mainframe applications

Zero Footprint Outside z/OS: The ZOWE server runs entirely within the z/OS environment:
* No need for external clients or servers
* Reduced infrastructure complexity
* Enhanced security by keeping operations within the mainframe
* Lower total cost of ownership

The ZOWE Ecosystem
ZOWE is more than just a tool - it's an ecosystem that includes:
CLI tools for command-line operations
API Mediation Layer for service discovery and API gateway functionality
Web UI for intuitive, graphical interactions with z/OS
By leveraging these components, organizations can modernize their mainframe operations, improve developer productivity, and bridge the gap between legacy systems and contemporary IT practices.

The home of ZOWE is at [www.zowe.org](https://www.zowe.org/)

## IBM offers:
A growing range of zowe-based tools for Db2 z/OS, based on the IBM Unified Management Server for z/OS, and the IBM Db2 Administration Foundation.

## This github repository offers:
A structured set of worked examples for using the zowe-based DB2 tooling, at the pages linked below....

Deployment Worked Examples
* [Deploying ZOWE](https://github.com/zeditor01/using_zowe/blob/main/docs/deploying_zowe.md)   
* [Deploying Unified Management Server for z/OS and Db2 Administration Foundation](https://github.com/zeditor01/using_zowe/blob/main/docs/deploying_ums_and_daf.md)
* [Configuring SQL Tuning abd DDL Services within Db2 Administration Foundation](https://github.com/zeditor01/using_zowe/blob/main/docs/configure_daf.md) Not yet written    
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

## The Destination 

Review what this new infrastructure allows us to do....

1. Easily view and understand the database objects in your Db2 Subsystem(s).
2. Manage the health of these database objects by utilising RTS health indicators to drive Db2 houskeeping automatically.
3. ....
4. Articulate the capabilities of the stack in terms that represent business value




