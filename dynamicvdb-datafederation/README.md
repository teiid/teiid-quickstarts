Dynamicvdb-datafederation is the 'Hello World' example for Teiid.  


This will demonstrate how to federate data from a relational data source with a text file-based data source.
This example uses the H2 database, which is referenced as the "accounts-ds" data source in the server, 
but the creation SQL can be adapted to another database if you choose.

Note:  this example provides the base setup for which other quick starts depend upon.

### Steps to setup and run the quickstart ###
These can be done either manually (see Setup manually) or using maven (see Setup using the JBoss AS Maven plugin) 

System requirements
-------------------

If you have not done so, please review the System Requirements (../README.md)

####################
#   Setup
####################

Setup can be done either manually (see Manual Setup) or using maven (see Setup using the JBoss AS Maven plugin) 


#########################################
### Manual setup
#########################################

1)  Start the server

	Open a command line and navigate to the "bin" directory under the root directory of the JBoss server

	For Linux:   ./standalone.sh -c standalone-teiid.xml	
	for Windows: standalone.bat -c standalone-teiid.xml

2)  Copy teiid support files
	
- Copy the "teiidfiles" directory to the $JBOSS_HOME/ directory

	The src/teiidfiles directory should contain:
	(1) customer-schema.sql
	(2) customer-schema-drop.sql
	(3) data/marketdata-price.txt
	(4) data/marketdata-price1.txt
	
when complete, you should see $JBOSS_HOME/teiidfiles

3) Setup the h2 datasource and file resource adapter

-  run the following CLI script

	-	cd to the $JBOSS_HOME/bin directory
	-	execute:  ./jboss-cli.sh --connect --file={path}/dynamicvdb-datafederation/src/scripts/setup.cli 

4)  Teiid Deployment:

Copy the following files to the $JBOSS_HOME/standalone/deployments directory

     (1) src/vdb/portfolio-vdb.xml
     (2) src/vdb/portfolio-vdb.xml.dodeploy

You should see the server log indicate the VDB is active with a message like:  TEIID40003 VDB Portfolio.1 is set to ACTIVE

5)  Open the admin console to make sure the VDB is deployed

	*  open a brower to http://localhost:9990/console 	

6)  See "Query Demonstrations" below to demonstrate data federation.


#########################################
### Setup using the JBoss AS Maven plugin
#########################################

1) Start the server

	Open a command line and navigate to the "bin" directory under the root directory of the JBoss server

	For Linux:   ./standalone.sh -c standalone-teiid.xml	
	for Windows: standalone.bat -c standalone-teiid.xml
	
2) setup the datasource

    * `mvn -Psetup-datasource jboss-as:add-resource` 
	
3) setup the resource adapter

    * `mvn -Psetup-rar jboss-as:add-resource`

* previously, a server restart was required at this point, but it is no longer required.
	
5) copy the vdb and teiidfiles support files

    *  `mvn install -Pcopy-files`

6)  Open the admin console to make sure the VDB is deployed

    *  open a browser to http://localhost:9990/console 	

7)  See "Query Demonstrations" below to demonstrate data federation.


##################################
#  Undeploy artifacts
##################################


1)  To undeploy the Teiid VDB, run the following command:

	*  mvn package -Pundeploy-vdb

NOTE: There currently isn't a JBoss AS plugin option for undeploying the rar and datasource that were
		setup in the configuration.  This will need to be done manually.	

#########################################
### Query Demonstrations
#########################################	

==== Using the simpleclient example ====

1) Change your working directory to "${quickstart.install.dir}/simpleclient"

2) Use the simpleclient example to run the following queries:

Example:   mvn install -Dvdb="portfolio" -Dsql="example query"


#################
# Example queries:
#################

Example 1 queries the relational source

Example 2 queries the text file-based source

Example 3 performs a join between the relational source and the text file-based source.  The files returned from the getTextFiles procedure are passed to the TEXTTABLE table function (via the nested table correlated reference f.file).  The TEXTTABLE function expects a 
text file with a HEADER containing entries for at least symbol and price columns. 

Example 4 Issue query that contains a NATIVE sql call that will be directly issued against the H2 database.  This is useful if the function isn't supported by the translator (check the documentation for the types of translators that support NATIVE sql).   Note that the translator override in the vdb xml enabling support for native queries has to be set.

1	select * from product
2	select stock.* from (call MarketData.getTextFiles('*.txt')) f, TEXTTABLE(f.file COLUMNS symbol string, price bigdecimal HEADER) stock
3.	select product.symbol, stock.price, company_name from product, (call MarketData.getTextFiles('*.txt')) f, TEXTTABLE(f.file COLUMNS symbol string, price bigdecimal HEADER) stock where product.symbol=stock.symbol
4 	select x.* FROM (call native('select Shares_Count, MONTHNAME(Purchase_Date) from Holdings')) w, ARRAYTABLE(w.tuple COLUMNS "Shares_Count" integer, "MonthPurchased" string ) AS x


