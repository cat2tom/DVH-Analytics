# LiveFreeOrDICOM
DICOM to SQL DVH Database

This code is intended for Radiation Oncology departments to build a SQL database of DVH's from DICOM files (Plan, Structure, Dose).
This is a work in progress.  This file will eventually contain instructions for an end-user.


### SQL Database format for this project
This code is being built assuming the database is MySQL.  There will be a master table called 'Plans' and 'DVHs'
in the database 'DVH'.  The 'Plans' table contains the following data:

Field | Type
----- | ----
MRN | varchar(12)
PlanID | tinyint(4) unsigned zerofill
Birthdate | date
Age | tinyint(3) unsigned
Sex | char(1)
CTSTudyDate | date
RadOnc | varchar(3)
TxSite | varchar(100)
RxDose | float
Fractions | tinyint(3) unsigned
Modality | varchar(20)
MUs | int(6) unsigned
PlanUID | varchar(19)

PlanID is based on the plans in the database currently associated with the MRN (e.g., no other plans with PatientUID then PlanID = 0001).  
PlanUID is  MRN + PlanID (e.g., MRN = 000111222333, PlanID = 0005, then PlanUID = 0001112223330005.

DICOM files do not explicitly contain prescriptions or treatment sites.  This code requires that the plan name be equal to the Tx Site.
The prescription is parsed from a point stored in the DICOM RT Structure file with a name called 'rx: ' + [total dose] + 'Gy'. 
This point will have to be added by the user in their planning system prior to DICOM export.  If this point is not found, the code will
assume the Rx Dose is equal to the max point dose.


The 'DVHs' table has one row per ROI and follows this format:

Field | Type
----- | ----
PlanUID | varchar(19)
ROIName | varchar(50) 
Type | varchar(20) 
Volume | double      
MinDose | double      
MeanDose | double      
MaxDose | double      
DoseBinSize | float       
VolumeString | mediumtext  
VolumeUnits | varchar(20) 


All data is populated from a combination of DICOM RT strucutre and dose files using dicompyler code.  VolumeString is a comma separated
value string, when parsed generates a vector for the DVH y-values.  The x-values are to be generated as a vector of equal length to the
y-axis with equally spaced values based on the DoseBinSize (e.g., VolumeString = '100,100,90,50,20,0' and DoseBinSize = 1.0 then
x-axis vector would equal [0.5 1.5 2.5 3.5 4.5 5.5]).

### Code organization
*DICOM_to_Python*  
This code contains functions that read dicom files and generate python objects containing the data required for input into the
SQL database.  There is no explicit user input.  All data is pulled from DICOM files.  The few values that DICOM files to not explicitly
record are entered in the Plan Name assuming the format '[Tx Site] [fractions] x [Dose]Gy.

*SQL_Tools*  
This code handles all communication with SQL with MySQL Connector.  No DICOM files are used in this code and require the python objects
generated by DICOM_to_Python functions.

*DICOM_to_SQL*  
This has the simple objective of writing to SQL Database with only the DICOM file paths as the input.
Currently this code is the least developed as it depends heavily on the previous two codes.

### To Do List
* Write SQL_Tools fuctions to
  * determine PlanUID for current DICOM import
  * remove a plan row from PatientPlans master table

* Write DICOM pre-import validation function
  * Ensure proper data is populated in DICOM files prior to import
  * Input will be a folder, function will find file path for all needed DICOM files

* Adjust Connect_to_SQL() from SQL_Tools to read a config file for SQL server login information

* Convert this to do list into a markdown task list


# References
Built on these Python libraries

pydicom  
https://github.com/darcymason/pydicom

dicompyler  
https://github.com/bastula/dicompyler

