## Data integration based on time stamp using SSIS

To operate a data integration process between two database systems, there will be two methods to do this:
The first method is to do a full data integrastion. Thats means the whole data in database A will be copied and send to database B. Normally, thiis kind of integration process is used for one time only at themost first database integration process.

The second method is a partial data intefgration thats operate between two database systems depending on a condition. However, in this method, only a pointed records inside the database A will be integrated with database B. As an example, the daily fingerprint records for the time attendance system thats required to integrate the daily records only. These daily records can be assigned for a data integration uses using one of the follwing keys:

The first key is the primary key. SSIS has an ability to integrate a records are in database A, but not available yet in database B. By using the "Lookup" tool in SSIS, the data integration service has the ability to make a comparasion between database A with database B and integrate the only missing data thats not available in database B. The "lookup" tool is useless in case if the table is running without a primary key.

The second key is by using the time stamp. By using s select statement thats pointing to a particular datetime, only the records whom matching with the selected datetime will be integrated and copied from database A to database B. This key is a backup for the first one that can be used with a less-primary key column in the selected table(s), but on the other hand, the fixed code of the select statement is required to change the datetime zone ( x> Datetime, x< Datetime, x> datetime >=y) which caused a mess thats the select statement needs to be updated to fetch a particular rows.

The Scenario will be going as follow:
The sources database is SQL server.
The destination database is oracle.
The selected rows are pointed regarding a time stamp column of the same table. 

The goal of the scenario: Back to the keys before that both of them are showing a good solution for the daily database integration. But in case of server failuer thats leads the SSIS job to be unavilable for several days, next time the SSIS job is going up and running, the job will etch the records of the same day, or the SQL statment has to be updated manually to cover the missing days.

Scenario steps:

Step 1: Create the tables
SQl server:

create or replace table source_tbl
(
  ID int identity(1,1) primary key, //Its not required to be a primary key in all tables.
  textclm nvarchar(30) not null,
  actiondate datetime not null
)

Oracle
create table target_tbl
(
  ID number (20) primary key,
  textclm varchar2(30) not null,
  actiondate datetime not null
)

Now destination and source tables are ready, but we still need to create one more table in SQL server side. This table will be used as a log table thats the last record which been integrated with oracle will be inserted in this table so that next time the SSIS job is run again, it willstart read the rows thats re greater than this record.

In SQL server

create table dailyLogs
(
	id int IDENTITY(1,1) NOT NULL,
	LastAction datetime NOT NULL,
	CreatedDate datetime NOT NULL,
)
All tables are ready for the next step

Step two: Design the SSIS jobs

Open visual studio 2019 or 2022 as an administrator and create a new sql server integration service project.
control flow
  From the SSIS toolbox in the right, drag and drop a data flow task.
  double click on the data flow task

Data flow
For SQL server connection
  In the connection Managers section, right click in there and select "New Connection"
  From the Menu, select "OLEDB" connection manager.
  Press on "New"
  Add your server information, username and password, and the select the database thats containing the table thats you created already.
  test you connection and press "OK"

For Oracle connection
  Download and install the "Microsoft connector for Oracle" over this link https://www.microsoft.com/en-us/download/details.aspx?id=104113
  In the connection Managers section, right click on it and select "New Connection"
  Select "Oracle conection Manager", and write down the connection name, tns service name, username and password, then finally test the connection and click "OK".
  
The work area
   Drag and drop "Source assistant". This one is for fetching the data form SQL Server table.
  	Select the connection manager thats you created before, is SQl server connection.
  	Click Ok, then double click on the new created box in the working area.
  	From Data access mode select "SQl command"
  	Add this SQl statment in the Sql command area.
  	" SELECT *
    	FROM 
    	source_tbl
    	where
   	actiondate > (select top(1) LastAction from dailyLogs order by CreatedDate desc)
    	order by  
    	actiondate desc "
   
   	The select statement aboce will select all the columns in the "source_tbl" thats the timestamp is greater than the on in table dailylogs.

   Drag and drop "Oracle destination" from the "SSIS Toolbox" side menu, and open it.
   	Select the connection from the "Oracle connection manager", the oracle connection thats you did it before.
	Select the table name.
	Don forget to match a line between "Source assistance" and "Oracle destination' in order of correct the mapping between the tables.
	
Now everything is done, lets go back to the "Control view", we need to create a variable.
The aim of this variable is to hold a value between "SQL Tasks"
In the "Variables menu" select "Add variable"
	Name: LatestDaily
	Scope: Pakage1 "Remember, the variable is running on the top of the bacjege only, not the whole project".
	Data type: Datetime
	Value: current day

   Drage and drop "Execute SQL task" from "SSIS Toolbox" and open it.
   Set the "ResultSet" to "Single row"
   Set the "Connection Type" to be "OLEDB".  
   Set the "Connection" o be the SQL server connection you created before.
   In the "SQLStatement" paste this command
   "select top(1) actiondate
   FROM
   source_tbl
   order by actiondate desc"
   
   Click "OK"
   
   From the left side menu click on "Result Set"
   Click on "Add".
   Set the "Result Name" to 0 and Variable Name to the variable thats you created before is "LatestDaily", then click "OK"
   
   The aim of the "Execure SQL task" above is to select the latest record in the source table and set the single result to the variable.
   
   One last thing thats we need is to create another "Execute SQL Task1", drag, drop and open it.
   Set the "ResultSet" to None
   Set the "ConnectionType" to OLEDB
   Set the "Connection" to sql connection that you created before.
   In the SQL statement, add the comand:
   "insert into dailyLogs
   values
   (?,getdate());"
   
   Click "OK"
   
   On the left side menu, click on "Parameter Mapping"
   Click on "Add"
   set the variable Name to the same variable you created before is "LatestDaily"
   Direction to Input
   Datatype to DATE
   ant the rest to 0.
   Click "OK" 
   

Now you have a three actions thats must connected to each other, the first one is the Data flow task, the second one is execute SQL task and the third one is execute SQL1.

The scenarion will run as follow
- The system will gather all data in source_tbl thats are greater than the very last record in dailyLogs table.
- The new data will find it way to the destination table.
- Then, in execute SQL task, the system will read the last enterd record in source_tbl and save the result in "LatestDaily" variable.
- The last step is running Execute SQL task1 to save the value in the dailylogs table
- The new value in the DailyLogs table will be the latest value for the next data integration job.

Thats all
Mohad Masadeh
   
   
   
  
   
   ID int identity(1,1) primary key, //Its not required to be a primary key in all tables.
  textclm nvarchar(30) not null,
  actiondate datetime not null
   


  
  
  
  

This page will run an SSIS job thats will use a select statemt to fetch a particular table rows and send them to a selected database. However, ou can use the [editor on GitHub](https://github.com/mbmasadeh/TimeStampDataMigration/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/mbmasadeh/TimeStampDataMigration/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
