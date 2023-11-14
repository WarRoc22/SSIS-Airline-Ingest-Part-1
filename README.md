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
Here is the following SQL script to the error checking stored procedure. This will error out records that have a duplicate PassengerID, missing PassengerID, missing FirstName, or missing LastName: 

USE [DBOR];
GO

SET ANSI_NULLS ON;
SET QUOTED_IDENTIFIER ON;
GO

CREATE OR ALTER PROCEDURE [dbo].[ProcessAirlineRaw]
    @LogExecutionFlag BIT = 1,
    @DebugStep SMALLINT = 0
AS

SET ARITHABORT ON;
SET XACT_ABORT OFF;
SET NOCOUNT ON;

CREATE TABLE #AirlineRaw
(
    AirlineRawID INT NOT NULL,
    PassengerID VARCHAR(200) NULL,
    FirstName VARCHAR(200) NULL,
    LastName VARCHAR(200) NULL,
    Gender VARCHAR(200) NULL,
    Age VARCHAR(200) NULL,
    Nationality VARCHAR(200) NULL,
    AirportName VARCHAR(200) NULL,
    AirportCountryCode VARCHAR(200) NULL,
    CountryName VARCHAR(200) NULL,
    AirportContinents VARCHAR(200) NULL,
    Continents VARCHAR(200) NULL,
    DepartureDate VARCHAR(200) NULL,
    ArrivalAirport VARCHAR(200) NULL,
    PilotName VARCHAR(200) NULL,
    FlightStatus VARCHAR(200) NULL,
    Errortext VARCHAR(MAX) NULL
);

INSERT INTO #AirlineRaw
(
    AirlineRawID,
    PassengerID,
    FirstName,
    LastName,
    Gender,
    Age,
    Nationality,
    AirportName,
    AirportCountryCode,
    CountryName,
    AirportContinents,
    Continents,
    DepartureDate,
    ArrivalAirport,
    PilotName,
    FlightStatus
)
SELECT
    ALR.AirlineRawID,
    ALR.PassengerID,
    ALR.FirstName,
    ALR.LastName,
    ALR.Gender,
    ALR.Age,
    ALR.Nationality,
    ALR.AirportName,
    ALR.AirportCountryCode,
    ALR.CountryName,
    ALR.AirportContinents,
    ALR.Continents,
    ALR.DepartureDate,
    ALR.ArrivalAirport,
    ALR.PilotName,
    ALR.FlightStatus
FROM
    dbo.AirlineRaw AS ALR;

WITH AirlineDupe AS
(
    SELECT
        PassengerID,
        ROW_NUMBER() OVER (PARTITION BY PassengerID ORDER BY PassengerID) AS AirlineDupe
    FROM
        #AirlineRaw
)
UPDATE lr
SET
    ErrorText = COALESCE(lr.Errortext, '') + 'Duplicate PassengerID in file.'
FROM
    #AirlineRaw AS lr
INNER JOIN AirlineDupe AS ad ON ad.PassengerID = lr.PassengerID
WHERE
    ad.AirlineDupe > 1;

UPDATE #AirlineRaw
SET
    Errortext = COALESCE(Errortext, '') + 'First Name is missing'
WHERE
    FirstName IS NULL;

UPDATE #AirlineRaw
SET
    Errortext = COALESCE(Errortext, '') + 'Last Name is missing'
WHERE
    LastName IS NULL;

UPDATE #AirlineRaw
SET
    Errortext = COALESCE(Errortext, '') + 'PassengerID is missing'
WHERE
    PassengerID IS NULL;

INSERT INTO dbo.AirlineError
(
    PassengerID,
    FirstName,
    LastName,
    Gender,
    Age,
    Nationality,
    AirportName,
    AirportCountryCode,
    CountryName,
    AirportContinents,
    Continents,
    DepartureDate,
    ArrivalAirport,
    PilotName,
    FlightStatus,
    ErrorText
)
SELECT
    ALR.PassengerID,
    ALR.FirstName,
    ALR.LastName,
    ALR.Gender,
    ALR.Age,
    ALR.Nationality,
    ALR.AirportName,
    ALR.AirportCountryCode,
    ALR.CountryName,
    ALR.AirportContinents,
    ALR.Continents,
    ALR.DepartureDate,
    ALR.ArrivalAirport,
    ALR.PilotName,
    ALR.FlightStatus,
    ALR.ErrorText
FROM
    #AirlineRaw AS ALR
WHERE
    ALR.Errortext IS NOT NULL;

INSERT INTO dbo.AirlineRepository
(
    PassengerID,
    FirstName,
    LastName,
    Gender,
    Age,
    Nationality,
    AirportName,
    AirportCountryCode,
    CountryName,
    AirportContinents,
    Continents,
    DepartureDate,
    ArrivalAirport,
    PilotName,
    FlightStatus
)
SELECT
    ALR.PassengerID,
    ALR.FirstName,
    ALR.LastName,
    ALR.Gender,
    ALR.Age,
    ALR.Nationality,
    ALR.AirportName,
    ALR.AirportCountryCode,
    ALR.CountryName,
    ALR.AirportContinents,
    ALR.Continents,
    ALR.DepartureDate,
    ALR.ArrivalAirport,
    ALR.PilotName,
    ALR.FlightStatus
FROM
    #AirlineRaw AS ALR
WHERE
    ALR.Errortext IS NULL;

# SSIS Control Flow
In the Control flow, I have the following components: 

Execute SQL Task - to truncate the raw table for every run. 

Data Flow task - To get data from the Excel file to insert into the raw table in SSMS 

Execute SQL task - This will run the error checking stored procedure with data that is in dbo.AirlineRaw. If the data passes the error checking, it will go to dbo.AirlineRepository. If it does not pass error checking it will go to dbo.AirlineError. 

File System task - This will move the file excel file to the Archive location directory that I created 

Foreach Loop Container - This will look through the file directory and grab any files to ingest that have Airline in their name. 

![CF1](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/d1e170d0-e762-49f3-aa59-91eb164e2141)

![CF2](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/7c9629b8-c3b4-4fdb-9be9-3db4dfe7e93e)

![CF3](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/aa810b36-ca2e-4e85-be11-44ce400f4bfd)

![CF4](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/a7b7d4cd-4190-47aa-b383-31f56de0d47b)

![CF5](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/56c06a11-a495-4472-8b16-a60cc9cf0925)

![CF6](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/c0387b81-1259-4269-810e-f837f1ce7766)

![CF7](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/5459f237-62de-4d21-989f-b2bc8d04726f)

![CF8](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/9a99e6c1-e0e8-41f8-941a-9cfc4453bf65)

# SSIS Data Flow
In the Data Flow I have the following components: 

Excel Source - Using this to extract data from the excel document 

Data Conversion - using this to convert the data types of the columns from Unicode String to String which is compatible with SSMS 

Row Count - Using this to get a count of the number of rows 

OLE DB Destination - Using this to load data into dbo.AirlineRaw

![DF1](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/4216c6b9-7a45-44c0-b642-619ca1a37b78)

![DF2](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/9cb71872-9ae0-4e2a-8731-d9e929deb002)

![DF3](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/cfebcaf5-58d9-4161-b3e7-0604a5473c50)

![DF4](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/bc6f777a-d927-42e9-911f-5c96facd23c1)

![DF5](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/aeeabed5-3f3b-4788-b694-5fa9564923b7)

![DF6](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/1c2d679f-dd16-45c1-968b-491130bb76e7)

![DF7](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/08db5d2e-515c-418d-a208-0806959a7cf8)

# Variables Used
These are the variables that I used in SSIS:

![V1](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/596bb316-b7e9-4b99-9e9a-3f59bd74febb)

# Tables and Directory Before Running the SSIS Package
![B1](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/84db649c-1beb-44cc-ab94-fc1c8dcf809f)

In the Airline directory, I have an excel file ready to be ingested:
![B2](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/e61a64a6-279a-4d66-b999-fc9f0fecfa9a)

No file currently in the Archive location: 
![B3](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/79e44e2a-bb05-4277-90be-1c670550e59b)

# Tables and Directory After Running the SSIS Package
![A1](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/3b3915a9-e7f3-4f8e-a775-f9e405ed570a)

![A2](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/12fbbb85-c47e-4e88-9ac5-1091d59d319b)

All records made it from the excel sheet to dbo.AirlineRaw  
![A3](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/5a0e5747-0a73-4c55-98f8-5226b75801e4)

These are the records that made it to dbo.AirlineError. These are the records that did not make it pass the error checking procedure. The ErrorText column gives a description of why. 
![A4](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/740c8efc-f79e-4aa1-85de-5eaee375cb63)

These are the records that made it to dbo.AirlineRepository. These are the records that made it through the error checking stored procedure. 
![A5](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/d653a5d7-679b-4e4f-be13-cb27d0b69608)

In the Airline directory the file now has moved to the archived folder since the SSIS Package was successfully completed:
![A6](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/42850167-87ff-47f5-a151-e161c731e21d)

The file is now in the Archive location: 
![A7](https://github.com/WarRoc22/SSIS-Airline-Ingest-Part-1/assets/148729293/1812cb4a-567f-465f-93ba-1053c23dc9eb)

# Deploying SSIS Package to SSMS
