# SSIS-Airline-Ingest-Part-1
Overview
This project implements an end-to-end data ingestion process using SQL Server Integration Services (SSIS) and SQL Server Management Studio (SSMS). The primary objective is to extract data from Excel files, perform error checking, and store the validated data in a SQL Server database.

SQL Server Setup
Data Models/Tables
dbo.AirlineRaw

A table designed to ingest all data from the Excel file without any constraints on data types.

dbo.AirlineError

This table stores data that did not pass the error checking stored procedure.

dbo.AirlineRepository

The final destination for validated data after passing through the error checking stored procedure.

Error Checking Stored Procedure
An error checking stored procedure identifies and records records with:
Missing First Name
Missing Last Name
Duplicate PassengerID
Missing PassengerID
SSIS Package Configuration
SSIS/Visual Studio Setup
AirlineIngest Package
Control Flow Components:

Execute SQL Task: Truncates the raw table before each run.
Data Flow Task: Extracts data from the Excel file and inserts it into the raw table in SSMS.
Execute SQL Task: Runs the error checking stored procedure on data in dbo.AirlineRaw, directing valid data to dbo.AirlineRepository and invalid data to dbo.AirlineError.
File System Task: Moves the Excel file to the Archive location directory.
Foreach Loop Container: Looks through the file directory and ingests any files with "Airline" in their name.
Data Flow Components:

Excel Source: Extracts data from the Excel document.
Data Conversion: Converts column data types from Unicode String to String for compatibility with SSMS.
Row Count: Provides a count of the number of rows.
OLE DB Destination: Loads data into dbo.AirlineRaw.
