# :computer: Crime Analytics - SQL for Beginners
## :globe_with_meridians: Set up your practice database
SQL (Structured Query Language) is how crime analysts ask questions about a database and this guide teaches users how to use SQL to analyze a fictional offender and incident-report data. 
This guide includes six mini lessons, then practice problems with an answer key. Every query below was run against the fictional database and the results shown are the real output.

## :file_folder: What you need
- A free tool that connects to SQL Server, such as SQL Server Management Studio (SSMS) or Azure Data Studio, also a SQL Server instance you can write to.
- The practice script ```cj_practice_sqlserver.sql```. Open the file in a query window and run it once. Doing so will create a database called ```cj_practice``` and builds and fills all four tables.
- After that, open a new query window. Make sure that ```cj_practice``` is the selected database (or start with USE ```cj_practice;```), paste a query and press execute (F5).

This guide uses SQL Server's dialect, T-SQL. PostgreSQL, MySQL and SQLite read almost the same. The differences are small and flagged where they matter (mainly limiting rows with ```TOP``` versus ```LIMIT```, and date functions). 

## :open_file_folder: The Data
Everything in this database is randomly generated. No row describes a real person, case or agency.

| Table | One row is | Rows | Columns |
| --- | --- | --- | --- |
| Offenders | One person | 60 | offender_id, first_name, last_name, birth_date, sex, supervision_type (Parole, Probation, None), risk_level (Low, Medium, High, or blank), prior_convictions, home_district |
| incidents | One reported incident | 200 | incident_id, report_date, offense_type, severity, district, location_type, weapon_involved (1 = yes, 0 = no), status, officer_id |
| arrests | One person arrested for one incident | 103 | arrest_id, incident_id, offender_id, arrest_date, charge, disposition |
| officers | one officer | 15 | officer_id, last_name, rank, district |

The tables connect through ID columns: arrests.incident_id points to incidents, arrests.offender_id points to offenders, and incidents.officer_id points to officers. The arrests table is the bridge between people and incidents, which matters in Lesson 5.

## :exclamation: Five ground rules

   1. End each query with a semicolon. SQL Server does not always require it, but it is good practice and it needed before a ```WITH``` clause.
   2. Text values go in single quotes: ```Burglary```. Numbers do not ```1```.
   3. Keywords are not case sensitive, but writing them in capitals makes queries easier to read.
   4. ```--``` starts a comment that SQL ignores.
   5. Clauses always appear in this order: ```SELECT``` (with ```TOP``` if you want to limit rows), ```FROM```, ```WHERE```, ```GROUP BY```, ```HAVING```, ORDER BY```.

# :notebook: Lesson 1: 
# ```SELECT``` and ```FROM``` - Looking at the data
Every query starts with two questions: which columns do you want (```SELECT```), and from which table (```FROM```)?
```SQL
SELECT TOP 5 first_name, last_name, risk_level
FROM offenders;
```

| first_name | last_name | risk_level |
| --- | --- | --- |
| Kevin | Ibarra | Medium |
| Lena | Turner | Low |
| Carlos | Reyes | NULL |
| Brianna | Vega | High |
| Omar | Underwood | Medium |

```TOP 5``` stops the output at five rows, a good habit when exploring a table with thousand of records. Without an ```ORDER BY``` (Lesson 3), SQL Server does not promise which five rows come back, so yours may differ from the table. ```NULL``` means the value is missing (here, no risk level was assigned). NULL is not the same as zero or an empty word, and Lesson 2 shows how to handle it.

## Select everything with ```*```
```SQL
SELECT TOP 5 *
FROM uncidents;
```

The star returns every column. It is handy for a first look at a table, but in real work name the columns you need.

## Rename columns with ```AS```
An alias gives a column a cleaner name in your results, which matters when the output goes into a report or dashboard. Giving columns an alias is also done for organizational purposes.
```SQL
SELECT TOP 3 first_name AS [First Name],
   prior_convictions AS priors
FROM offenders;
```

## Remove repeats with ```DISTINT```
```DISTINCT``` returns each value once. Use it to see what categories a column holds.
```SQL
SELECT DISTINCT district
FROM incidents;
```

| district |
| --- |
| South |
| Central |
| East |
| West |
| North |

To count the categories instead of listing them, wrap the column: 
```sql
SELECT COUNT(DISTINCT offense_type)
FROM incidents;
```
This returns a count of 10.

### Try it yourself!
1. Show ```offense_type```, ```report_date```, and ```district``` for the first 10 incidents.
2. Lis every distinct ```location_type``` in incidents.
3. Show all columns for the first 3 officers.

Answers are in the answer key.

# :notebook: Lesson 2:
# ```WHERE``` - filtering to the rows you need
```WHERE``` keeps only the rows that meet a condition, which is how you narrow 200 incidents down to the ones you care about.
```sql
SELECT TOP 5 incident_id, report_date, district
FROM incidents
WHERE offense_type = 'Burglary'
ORDER BY incident_id;
```
| incident_id | report_date | district |
| --- | --- | --- |
| 13 | 2025-10-01 | West |
| 17 | 2025-11-30 | North |
| 39 | 2025-09-02 | North |
| 55 | 2025-08-21 | East |
| 81 | 2025-05-21 | North |

Swap ```TOP 5``` and the column list for ```COUNT(*)``` and the same filter shows there are 18 burglaries in the data. Spelling and spaces must match; capitals usually do not matter in SQL Server's default setup.

## Combine conditions with AND, OR and parentheses
```AND``` requires all conditions to be true for a record to be included. ```OR``` requires at least one condition to be true.
```SQL
SELECT incident_id, report_date, offense_type
FROM incidents
WHERE district = 'North'
  AND weapon_involved = 1;
```
This returns 6 incidents; the first rows are:
| incident_id | report_date | offense_type |
| --- | --- | --- |
| 20 | 2025-03-25 | Weapons Violation |
| 124 | 2025-07-22 | Weapons Violation |
| 125 | 2025-05-11 | Simple Assault |
| 139 | 2025-10-11 | Weapons Violation |
| 161 | 2025-08-18 | Larceny |

When you mix ```AND``` with ```OR```, SQL evaluates ```AND``` before ```OR```. Always use parentheses to explicitly group your conditions so the database executes your intended logic:
```sql
WHERE district = 'North' AND (offense_type = 'Robbery' OR offense_type = 'Burglary')
```
is a different question from the same line without the parentheses.

## Handy filter shortcuts
| Tool | What it does | Example | Rows |
| --- | --- | --- | --- |
| ```IN``` | Matches any value in a list | ```WHERE offense_type IN ('Robbery', 'Aggravated Assault')``` | 23 |
| ```BETWEEN``` | Matches a range, ends included | ```WHERE report_date BETWEEN '2025-06-01' AND '2025-08-31'``` | 51 |
| ```LIKE`` | Matches a text pattern; ```%``` means any characters | ```WHERE offense_type LIKE '%Assault%``` | 36 |
| ```!=``` or ```<>``` | Not equal | ```WHERE status != 'Open'``` | |

The ```BETWEEN``` example is the summer 2025 incidents. In the practice database ```report_date``` is a ```DATE``` column; write date values as ```'YYYY-MM-DD'``` and SQL Server reads them correctly.
