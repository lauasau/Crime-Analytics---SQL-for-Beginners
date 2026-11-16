# :computer: SQL for Beginners
SQL (Structured Query Language) is how crime analysts ask questions about a database and this guide teaches users how to use SQL to analyze a fictional offender and incident-report data. 
This guide includes six mini lessons, then practice problems with an answer key. Every query below was run against the fictional database and the results shown are the real output.

What you need
- A free tool that connects to SQL Server, such as SQL Server Management Studio (SSMS) or Azure Data Studio, also a SQL Server instance you can write to.
- The practice script ```cj_practice_sqlserver.sql```. Open the file in a query window and run it once. Doing so will create a database called ```cj_practice``` and builds and fills all four tables.
- After that, open a new query window. Make sure that ```cj_practice``` is the selected database (or start with USE ```cj_practice;```), paste a query and press execute (F5).

This guide uses SQL Server's dialect, T-SQL. PostgreSQL, MySQL and SQLite read almost the same. The differences are small and flagged where they matter (mainly limiting rows with ```TOP``` versus ```LIMIT```, and date functions). 

## :file_folder: The Data
Everything in this database is randomly generated. No row describes a real person, case or agency.

| Table | One row is | Rows | Columns |
| --- | --- | --- | --- |
| Offenders | One person | 60 | offender_id, first_name, last_name, birth_date, sex, supervision_type (Parole, Probation, None), risk_level (Low, Medium, High, or blank), prior_convictions, home_district |
| incidents | One reported incident | 200 | incident_id, report_date, offense_type, severity, district, location_type, weapon_involved (1 = yes, 0 = no), status, officer_id |
| arrests | One person arrested for one incident | 103 | arrest_id, incident_id, offender_id, arrest_date, charge, disposition |
| officers | one officer | 15 | officer_id, last_name, rank, district |
