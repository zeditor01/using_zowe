# Header

xxx
xxx





```

SQL Tuning Services fro Db2 z/OS V13
https://www.ibm.com/docs/en/db2-for-zos/13?topic=db2-sql-tuning-services

SQL Tuning Services consists of a set of SQL analysis and tuning features that are delivered as RESTful APIs. 
You use these features to analyze and tune SQL applications that work with Db2 for z/OS.

https://www.ibm.com/docs/en/db2-for-zos/13?topic=services-api-reference

No-charge : Visual Explain,  Statistics Advisor, Capture Query Environment,  (IBM Database Services Expansion Pack feature of Db2 Accessories Suite for z/OS). 
Billable : Path Advisor, Access Path Comparison, Index Advisor, and more (IBM Db2 Query Workload Tuner for z/OS 6.1).

Additionally, SQL Tuning Services functionality is integrated into the user interfaces of the following products:
IBM Db2 Administration Foundation for z/OS leverages the SQL Tuning Services APIs.
IBM Db2 for z/OS Developer Extension integrates the SQL Tuning Services APIs into a Microsoft Visual Studio Code development environment.

===

components and architecture
- SQL Tuning Services Server (running in USS)
- Repository Database 
- Target Db2 z/OS Database 
- Tuning Profiles 

=== 

installation roadmap

1. Allocate System Capacity - OK
2. Assign Userids to install, configure, administer SQL Tuning Services 
3. Configure Network Ports - take defaults 
4. Configure SQL Tuning Services Environment (USS env) and setup ID 
5. Install and Configure SQL Tuning Services 
6. Optionally - configure AT-TLS 
7. Optionally - install license
8. Setup Tuning Services Environment (repository DB, tuning profile, explain tables) 

===

installation pre-reqs 

1 cp or ziip, 4gb RAM, 5gb DASD
zos 
liberty server 
db2 zos v12 or later 
ICSF 
SMPE stuff 
Optional License 

===

Setup Userids and Permissions 

You must allocate or create these user IDs before you start to install SQL Tuning Services.

tms_setup_userid
- used to install, configure, and start SQL Tuning Services in UNIX System Services.
- $JAVA_HOME/bin defined in the $PATH environment variable in the user's profile
- The $_BPXK_AUTOCVT environment variable set to ON in the user's profile
- Permission to read and execute to the install_dir_zos directory or a directory similar to $TMS_HOME that's used by the SMP/E installation process.
- The $IBM_JAVA_OPTIONS environment variable set to the following value in the user's profile: -Dfile.encoding=UTF-8

db2_authid_R
- used by the USS Server to connect to the Db2 Tuning Repository DB
- CREATEDBA ON SYSTEM TO db2_authid_R
- CREATEIN ON SCHEMA IBMTMS TO db2_authid_R
- EXECUTE privilege on all of the packages that are listed in the DSN5RTRP sample job.

tms_userid
- used to login to the Server & do tuning work.
- connect privilege to to Repository DB 
- EXECUTE ON FUNCTION IBMTMS.CANVIEW TO tms_userid
- EXECUTE ON FUNCTION IBMTMS.CANADMINISTER TO tms_userid 

db2_authid_T
- used to connect to the target DB 
- EXECUTE privilege on the packages that are listed in the DSN5RTTG sample job.
- SQLADM 
- etc...

>>>
Use IBMUSER for both the Db2 authids
Use IBMUSER as the tms_setup_userid 
Create IBMTUNE as the tms_userid


===

Configure SQL Tuning Services for tms_setup_userid (IBMUSER) 
/bin/ulimit -t unlimited
/bin/ulimit -A 1048576
/bin/ulimit -M 2560

/bin/ulimit -a
core file         8192b
cpu time          unlimited
data size         unlimited
file size         unlimited
stack size        unlimited
file descriptors  520000
address space     1048576k
memory above bar  2560m


===

Install and Configure 

Step 1 - Install SQL Tuning Services

Install SQL Tuning Services on your z/OS system by following the instructions 
in the Program Directory for the product that you are installing: 
- the IBM Database Services Expansion Pack Program Directory 
or 
- the Db2 Query Workload Tuner for z/OS Program Directory.

/usr/lpp/IBM/db2tms/v2r1
...
_Dir      755   OMVSKERN         8192   tmsinstall
_Dir      755   OMVSKERN         8192   tmsservice
_Dir      755   OMVSKERN         8192   IBM

Two partitioned data sets are created to contain the following components:
hlq. SDSN5TSA - This data set contains the sample JCL.
hlq. SDSN5TDB - This data set contains the DBRMs.

... on ADCD it is 
'DSND10.ADSN5TBA'
'DSND10.ADSN5TZF'

Step 2 - Configure secure network communications

This step provides for https protocol. (can bypass this and use AT-TLS if you like)

Keyring & Self-Signed certificate.
- You can use an existing key ring if one is available on your system, 
- or you can generate a file-based certificate on the z/OS system that you are installing SQL Tuning Services on. 
- The certificate must be a SAN (Subject Alternative Name) or wildcard SSL certificate that allows the specification 
- of multiple domains or hosts. Make sure that your SSL certificate contains the IP addresses of your SQL Tuning Services system. 
- The certificate must be CA- or self-signed.


Step 3 - Edit tmsservice.config 

Copy the install_dir_zos/tmsinstall/tmsservice.config file into a writable directory (new_dir/tmsservice.config) 
and edit the following installation options:


Step 4 - Configure SQL Tuning services 

Configure SQL Tuning Services by running the tmsservice.sh script in the install_dir_zos/tmsinstall directory:
./tmsservice.sh new_dir/tmsservice.config


Step 5 - Create UDFs 

DSND10.ADSN5TBA(DSN5RUDF)

Step 6 - Bind Packages

DSND10.ADSN5TBA(DSN5NDRP)
DSND10.ADSN5TBA(DSN5NDTG)


Step 7 Grant privileges

DSND10.ADSN5TBA(DSN5RTRP)
DSND10.ADSN5TBA(DSN5RTTG)

Step 8 - Optional set USS permissions

Step 9 - JCL to start

DSND10.ADSN5TBA(DSN5STRT)


===

License
 
 
Locate the Db2 Query Workload Tuner license.jar file. 
After the SMP/E installation, the default location of this file is /usr/lpp/IBM/qwtz.

Create a license folder at the following location: 
wlp_user_dir/servers/server_name/license

where wlp_user_dir and server_name match the values that you specified for these parameters in tmsservice.config 
when you installed and configured SQL Tuning Services.

Copy the license.jar file to the license folder that you created.



===

Setting Up SQL Tuning Services Environment 


Step 1a - Generate a token for the temporary SQL Tuning Services username that you specified in installation and configuration step 4.


curl -X 'POST' \
  'https://service_ip:httpsport/tuningservice/v1/auth/tokens' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "userid": "ADMIN1",
  "password": "password"
}'


Step 1b - Create a connection to the SQL Tuning Services repository database 
by invoking the Repository Database Setup API and specifying the token that you created in the previous step.


curl -X POST 'https://service_ip:httpsport/tuningservice/v1/set_repo' \
--header 'Authorization: Bearer Bearer_token_for_temporary_username' \
--header 'Content-Type: application/json' \
--data-raw '{
  "credential": {
    "user": "{db2_authid_R}",
    "password": "{password}"
  },
  "host": "{Db2_host_R}",
  "location": "{location}",
  "port": "{port_number}",
  "sslConnection": "true",
  "sslTrustStoreLocation": "{safkeyring://racf-id/ring-id}",
  "sslTrustStorePassword": "password",
  "additionalProperties": {
    "sslTrustStoreType": "JCERACFKS"
  }
}'


Step 1c - Generate a new token for an SQL Tuning Services administrator ID by invoking the Authentication Service API.


curl -X 'POST' \ 
  'https://service_ip:httpsport/tuningservice/v1/auth/tokens' \ 
  -H 'Accept: application/json' \ 
  -H 'Content-Type: application/json' \ 
  -d '{ 
  "userid": "ADMIN1", 
  "password": "password" 
}'



Step 1d - Create the REpository DB 


curl -X 'POST' \
  'https://service_ip:httpsport/tuningservice/v1/repodb' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer Bearer_token_for_the_SQL_Tuning_Services_ID' \
  -H 'Content-Type: application/json' \
  -d '{
  "default_sqlid": "ADMIN1",
  "ix16kbufferpool": "BP16K2",
  "ix4kbufferpool": "BP2",
  "ix8kbufferpool": "BP8K2",
  "runddl": true,
  "storagegroup": "SYSDEFLT",
  "ts16kbufferpool": "BP16K1",
  "ts4kbufferpool": "BP1",
  "ts8kbufferpool": "BP8K1"
}'


Step 2 - Create a tuning profile to connect to the target Db2 for z/OS database that contains the SQL that you want to tune.

Step 3 - Create the EXPLAIN tables to store the EXPLAIN information that SQL Tuning Services relies on to tune SQL. 

Step 4 - IVP 

```
