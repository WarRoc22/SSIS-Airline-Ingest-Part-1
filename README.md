# SSIS-Airline-Ingest-Part-1
For this project, I'm implementing an end-to-end ingest process using SSIS and SQL. 

In SQL, I've created three data models/tables for where the ingest data from the Excel file will go. The three data models are:

dbo.AirlineRaw: This table is to get all the data into a table without worrying about missing any data. I have set all the data types for all the columns to Varchar(200) except for AirlineID, which is an identity column.

dbo.AirlineError: This table is used for data that did not make it past the error-checking stored procedure that I created.

dbo.AirlineRepository: This table is the final resting place for the data. This will have all data that made it past the error-checking stored procedure.

I've also created an error-checking stored procedure. The error-checking stored procedure will error out any record that has the following:

Missing First Name,
Missing Last Name,
Duplicate PassengerID,
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

# Excel Sheet to Ingest
The source where I got the Excel sheet to ingest is from Kaggle: 

[https://www.kaggle.com/datasets/iamsouravbanerjee/airline-dataset](url)

I then made changes to the Excel sheet like creating duplicates and taking out data from different columns. For this project, I used the modified version

Here is the original version: 

[Airline_20231001.xlsx](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/files/13343084/Airline_20231001.xlsx)

Here is the modified version: 

[Airline_20231113.xlsx](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/files/13343090/Airline_20231113.xlsx)

# Data Model Creation
Here is the following SQL script that I used to create the three data models: 

Use DBOR 

Go 
 
Set ANSI_NULLS ON; 

SET QUOTED_IDENTIFIER ON; 

Go 
 
IF NOT EXISTS (Select * 
		From INFORMATION_SCHEMA.TABLES As T 
		Where T.TABLE_SCHEMA = 'dbo' 
		And T.TABLE_NAME = 'AirlineRepository') 
Begin 
Create Table dbo.AirlineRepository 
( 
AirlineID Int IDENTITY(1,1) NOT NULL, 
PassengerID Varchar(30)  NOT NULL, 
FirstName Varchar(50)  NOT NULL, 
LastName Varchar(50)  NOT NULL, 
Gender Varchar(15)  NOT NULL, 
Age Int NOT NULL, 
Nationality Varchar(100) NOT NULL, 
AirportName Varchar(200) NOT NULL, 
AirportCountryCode Varchar(10)   NOT NULL, 
CountryName Varchar(200) NOT NULL, 
AirportContinents Varchar(10)   Not NULL, 
Continents Varchar(100) NOT NULL, 
DepartureDate Date NOT NULL, 
ArrivalAirport Varchar(10) NOT NULL, 
PilotName Varchar(200) NOT NULL, 
FlightStatus Varchar(50) NOT NULL, 
Constraint PK_dbo_Airline Primary Key Clustered (AirlineID) 
); 
End; 
GO 
 
 
IF NOT EXISTS (Select * 
		From INFORMATION_SCHEMA.TABLES As T 
		Where T.TABLE_SCHEMA = 'dbo' 
		And T.TABLE_NAME = 'AirlineRaw') 
Begin 
Create Table dbo.AirlineRaw 
( 
AirlineRawID Int IDENTITY(1,1) NOT NULL, 
PassengerID Varchar(200) NULL, 
FirstName Varchar(200) NULL, 
LastName Varchar(200) NULL, 
Gender Varchar(200) NULL, 
Age Varchar(200) NULL, 
Nationality Varchar(200) NULL, 
AirportName Varchar(200) NULL, 
AirportCountryCode Varchar(200) NULL, 
CountryName Varchar(200) NULL, 
AirportContinents Varchar(200) NULL, 
Continents Varchar(200) NULL, 
DepartureDate Varchar(200) NULL, 
ArrivalAirport Varchar(200) NULL, 
PilotName Varchar(200) NULL, 
FlightStatus Varchar(200) NULL, 
Constraint PK_dbo_AirlineRaw Primary Key Clustered (AirlineRawID) 
); 
End; 
GO 
 

IF NOT EXISTS (Select * 
		From INFORMATION_SCHEMA.TABLES As T 
		Where T.TABLE_SCHEMA = 'dbo' 
		And T.TABLE_NAME = 'AirlineError') 
Begin 
Create Table dbo.AirlineError 
( 
AirlineErrorID Int IDENTITY(1,1) NOT NULL, 
PassengerID Varchar(200) NULL, 
FirstName Varchar(200) NULL, 
LastName Varchar(200) NULL, 
Gender Varchar(200) NULL, 
Age Varchar(200) NULL, 
Nationality Varchar(200) NULL, 
AirportName Varchar(200) NULL, 
AirportCountryCode Varchar(200) NULL, 
CountryName Varchar(200) NULL, 
AirportContinents Varchar(200) NULL, 
Continents Varchar(200) NULL, 
DepartureDate Varchar(200) NULL, 
ArrivalAirport Varchar(200) NULL, 
PilotName Varchar(200) NULL, 
FlightStatus Varchar(200) NULL, 
ErrorText Varchar(1000)NULL, 
Constraint PK_dbo_AirlineError Primary Key Clustered (AirlineErrorID) 
); 
End; 
GO 



# Error Checking Stored Procedure
# SSIS Control Flow
# SSIS Data Flow
# Variables Used
# Tables and Directory Before Running the SSIS Package
# SSIS Package
# Tables and Directory After Running the SSIS Package
# Deploying SSIS Package to SSMS
