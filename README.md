# SSIS-Airline-Ingest-Part-1
• For this project I'm doing an end to end ingest process using SSIS and SQL. 

• In SQL I did the following: 
	• Created three data models/tables for where the ingest data from the excel file will go. The three data models is: 
		○ dbo.AirlineRaw - This table is to get all the data into a table without worrying about missing any data. I have set all the data types for all the columns to Varchar(200) except for AirlineID which is an identity column. 
		○ dbo.AirlineError - This table is used for data that did not make it pass the error checking stored procedure that I created. 
		○ dbo.AirlineRepository - This table is the final resting place for the data. This will have all data that made it passed the error checking stored procedure  
	• Created an error checking stored procedure.   
		○ The error checking stored procedure will error out an record that has the following 
			§ Missing First Name 
			§ Missing Last Name 
			§ Duplicate PassengerID 
			§ Missing PassengerID  
	• Deployed the SSIS package from SSIS to SSMS 
• In SSIS/Visual Studio I did the following: 
	• Created the AirlineIngest Package 
		○ In the Control flow I have the following components: 
			§ Execute SQL Task - to truncate the raw table for every run. 
			§ Data Flow task - To get data from the excel file to insert into the raw table in SSMS 
			§ Execute SQL task - This will run the error checking stored procedure with data that is in dbo.AirlineRaw. If the data passes the error checking, it will go to dbo.AirlineRepository. If it does not pass error checking it will go to 
                          dbo.AirlineError. 		
			§ File System task - This will move the file excel file to the Archive location directory that I created 
			§ Foreach Loop Container - This will look through the file directory and grab any files to ingest that  has Airline in its name.   
		○ In the Data Flow I have the following components: 
			§ Excel Source - Using this to extract data from the excel document
			§ Data Conversion - using this to convert the data types of the columns from Unicode String to String which is compatible with SSMS 
			§ Row Count - Using this to get a count of the number of rows 
			§ OLE DB Destination - Using this to load data into dbo.AirlineRaw 

