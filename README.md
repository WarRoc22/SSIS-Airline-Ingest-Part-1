# SSIS-Airline-Ingest-Part-1
For this project, I'm implementing an end-to-end ingest process using SSIS and SQL. 

In SQL, I've created three data models/tables for where the ingest data from the Excel file will go. The three data models are:

dbo.AirlineRaw: This table is to get all the data into a table without worrying about missing any data. I have set all the data types for all the columns to Varchar(200) except for AirlineID, which is an identity column.

dbo.AirlineError: This table is used for data that did not make it past the error-checking stored procedure that I created.

dbo.AirlineRepository: This table is the final resting place for the data. This will have all data that made it past the error-checking stored procedure.

I've also created an error-checking stored procedure. The error-checking stored procedure will error out a record that has the following:

Missing First Name
Missing Last Name
Duplicate PassengerID
Missing PassengerID
I've deployed the SSIS package from SSIS to SSMS.

In SSIS/Visual Studio, I did the following:

Created the AirlineIngest Package

In the Control flow, I have the following components:

Execute SQL Task: To truncate the raw table for every run.
Data Flow task: To get data from the Excel file to insert into the raw table in SSMS.
Execute SQL task: This will run the error-checking stored procedure with data that is in dbo.AirlineRaw. If the data passes the error-checking, it will go to dbo.AirlineRepository. If it does not pass error-checking, it will go to dbo.AirlineError.
File System task: This will move the Excel file to the Archive location directory that I created.
Foreach Loop Container: This will look through the file directory and grab any files to ingest that have Airline in their name.
In the Data Flow, I have the following components:

Excel Source: Using this to extract data from the Excel document.
Data Conversion: Using this to convert the data types of the columns from Unicode String to String, which is compatible with SSMS.
Row Count: Using this to get a count of the number of rows.
OLE DB Destination: Using this to load data into dbo.AirlineRaw.
