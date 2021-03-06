Version History/Changes for Denodo Scanner
------------------------------------------
2019/11/29 - v0.9.8.5
- bugfix java.lang.NullPointerException when wrapper/datasource objects failes (permission related)
  Note:  this fixes the NullPointerException - but there will be no external lineage for any object that we cannot execute a desc vql statement on
         see here:  https://community.denodo.com/docs/html/browse/7.0/vdp/administration/databases_users_and_access_rights_in_virtual_dataport/user_and_access_right_in_virtual_dataport/user_and_access_right_in_virtual_dataport#write-privilege
         

2019/10/25 - v0.9.8.4
- bugfix #32 NullPointerException after error in extractWrapper
- location property is now written for Tables with the denodo folder location (missing before), was only stored for views before

2019/10/18 - v0.9.8.3
- bugfix #28 - gracefully capture the following error:
					java.sql.SQLException: The user does not have WRITE privileges on ...
               modified the code to catch these errors and print an error summary (no stack trace) to the console (& debug log)

               Note 1:  this should not affect any relational (JDBC/ODBC) datasources.  however for datafile (DF) datasources
                        and 
                        it means that lineage back to the filesystem objects will be missing
               Note 2:  this is a restriction with Denodo (so not a scanner bug) 
                        where WRITE privileges are required to view the VQL for 
                        Data sources, derived views, base views, stored procedures, JMS listeners, web services and widgets
                        https://community.denodo.com/docs/html/browse/7.0/vdp/administration/databases_users_and_access_rights_in_virtual_dataport/user_and_access_right_in_virtual_dataport/user_and_access_right_in_virtual_dataport


2019/10/10 - v0.9.8.2
- bugfix #26 - views with a + character in the name were not processed properly
- bugfix #25 - datasource and wrapper objects with spaces in the name were not stored properly, lineage would be missing for views based on these objects
               regex patterns are used to extract the datasource & wrapper names
- bugfix #24 - extracting the vql (create statement) for some views failed to find the end of a view definitions ";" character.  previous versions were looking for );\n 
               the fixed version only looks for ;\n
- new feature - denodo database to database (schemas in the EDC relational model) are now linked via core.DataSetDataFlow relationship
                Note:  recursive relationships (where views are based on other views/tables in the same database) are not stored (can clutter the schema lineage diagram)


2019/08/08 - v0.9.8.1
- bugfix - lineage.csv had single quotes for columns in some situations

2019/08/05 - v0.9.8
- custom for delimited files (wrapper type=DF) created
- sql over-rides for base views (tables) is captured using com.infa.ldm.relational.ViewStatement
  lineage is not possible for these tables (see known limitations) 
  the scanner log (console) will print the count of "tables with sql statements:<n>" 
  and list each entry <db>.<basetable>
  to extract lineage a sql parser would be required
- datasource metadata is now collected during a 1st pass for each denodo database
  this is used to extract custom lineage for non JDBC/ODBC sources
- custom lineage for jdbc - now using correct column name from wrapper definition
  (was assuming column names were the same as what is used in the denodo base view)
- view sql statements were including all dependent views and datasource/wrappers
  only the definition for the view is stored now - reducing any truncation messages during import
  
  
Known Limitations:
1.	custom lineage is not possible for base views using SQL statements


2019/07/22 - v0.9.7
- all object .csv files are now prefixed objects-*.csv (needed for 10.2.2hf1+)
- changes for v10.2.2hf1 and later (for older versions, processing remains the same)
  - added new flag in .properties file "include.custlineage" - with a default of false
    when set to true - lineage will be generated in a seperate folder, lineage.csv will
    be included in the scanner output
  - when importing into EDC, using v10.2.2hf1+ - set the following:-
    ETL Resource=Yes  (etl really means the scanner has external lineage in this case)
    Auto Assign Connections - can be on or off, same as for any custom lineage resource
  - connections naming convention <denodo_database>.<denodo_connection_name>:<wrappertype>

2019/07/08 - v0.9.6
- re-factored view lineage processing column level
	- previous versions processed each column individually, 
	  for some larger databases that hava a lot of joins, these 
	  querys could take up to 4 seconds for each column
	- by re-factoring, we can get the same result by extracting 
	  lineage for all columns in a view (vs 1 at a time).  
	  For a large database, this reduced the scan time from 1h:20 to 9 mins
	- console output is more compact
- re-factored view lineage processing view level
	- older versions created some wrong links for interface views 
	  (links to the objects linked from the source of the interface)
- fixed Issues(github)
  https://github.com/Informatica-EIC/Custom-Scanners/issues
	- #8  - denodo: tables/views with $ character in the name fail for jdbc getColumns
	- #9  - denodo: tables with mixed case names - extract wrapper fails
	- #10 - denodo: GET_SOURCE_TABLE [STORED_PROCEDURE] [ERROR] for non jdbc wrappers
	- #12 - denodo: scanner raises errors when databases have mixed case names
	
- known issues / enhancements requested
	- #13 - denodo: custom lineage for flatfiles is not generated
			for delimited files, we should be able to generate lineage 
			all other datasource types (Except for JDBC) will not have any 
			custom lineage generated
			for any non-supported datasource types - you will see a message in the 
			scanner console/log
				wrapper type:{type} not yet supported
				
			
	
	