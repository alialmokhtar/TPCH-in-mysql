# TPCH-in-mysql

Implementing TPC-H into MYSQL
I am going to show how to implement TPC-H benchmark into MYSQL using Ubuntu.

prerequisite:
  * MYSQL server installed in Ubuntu OS.
  * Downloading DBGen zip file from http://www.tpc.org/tpch/spec/tpch_2_16_0.zip
  * unzipping the file
  * Navigating into dbgen folder
  

*****step one:

 We have to modify Makefile to fit our enviroment.
 $vi makefile
  then modify DATABASE, MACHINE, WORKLOAD like :
  
  ################
## CHANGE NAME OF ANSI COMPILER HERE
################
CC      = gcc
# Current values for DATABASE are: INFORMIX, DB2, TDAT (Teradata)
#                                  SQLSERVER, SYBASE, ORACLE
# Current values for MACHINE are:  ATT, DOS, HP, IBM, ICL, MVS, 
#                                  SGI, SUN, U2200, VMS, LINUX, WIN32 
# Current values for WORKLOAD are:  TPCH
DATABASE= SQLSERVER
MACHINE = LINUX
WORKLOAD = TPCH
########################

******step2
we need to modify tpcd.h as below:
find #ifdef SQLSERVER and modify as deinfe START_TRAN, define END_TRAN, define SET_ROWCOUNT, and deinfe SET_DBASE as it's shown below:

############################################
#ifdef  SQLSERVER
#define GEN_QUERY_PLAN  "set showplan on\nset noexec on\ngo\n"
#define START_TRAN      "BEGIN WORK;"
#define END_TRAN        "COMMIT WORK;"
#define SET_OUTPUT      ""
#define SET_ROWCOUNT    "limit %d;\n\n"
#define SET_DBASE       "use %s;\n"
#endif
###############################################

******steop 3
$make


*****step 4

generate the data using dbgen like:
$ ./dbgen -s 1
      -s scale factor 1, you can change it to whatever you want , depening on the wanted size of data.
******step 5
$ mysql -u root -p 
$ mysql> CREATE DATABASE tpch;
$ mysql> USE tpch;

******step 6
Creating tables using dss.ddl
$ mysql> \. dss.ddl
Loading tables with the generated data:


LOAD DATA LOCAL INFILE 'customer.tbl' INTO TABLE CUSTOMER FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'orders.tbl' INTO TABLE ORDERS FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'lineitem.tbl' INTO TABLE LINEITEM FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'nation.tbl' INTO TABLE NATION FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'partsupp.tbl' INTO TABLE PARTSUPP FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'part.tbl' INTO TABLE PART FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'region.tbl' INTO TABLE REGION FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE 'supplier.tbl' INTO TABLE SUPPLIER FIELDS TERMINATED BY '|';


Now adding primary keys to our tables using the script:
mysql> \. dss.ri

Then you can start running your tests in TPC-H database
