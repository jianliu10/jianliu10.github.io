---
layout: post
title:  "Case Study - automation enhancements in a retail data integration component in data warehouse"
date:   2019-2-22 00:00:00 -0500
categories: tech data-integration
---

# Background
Promotion is a key part of Merchandizing Sales and Marketing system. This time, the sales and marketing business wants to add 40 new promotion programs, change 10 existing promotion programs, and add 50 new promotion sub-programs.

I was tasked to enhance an 10+ years aged existing Promotion data integration component in a data warehouse system. The goal of the enhancement is not only to add new promotion programs and sub-programs for this one-time project, but also to automate the promotion data integration process in DWS.

This document describes the as-of state of existing promotion data integration component in DWS, and its enhancement design. 


# Abbreviations  

DWS - Data Warehouse System  
SAM - the Merchandizing Sales and Marketing transactional system  
PROMO-DI - the promotion data integration component in DWS  
PromoTree - the retail store promotion management system  
ORM - Online Order Management system  


# the as-of state of existing promotion data integration component

PROMO component in DWS provides data for a few key business functions:
- provide fact-dimensional data for Reports and Business Intelligence Analysis.
- provide flat store&item level promotional data for downstream retail store promotion management system. 
- provide flat corporate&item level promotional data for downstream Online Order Management system.


# The issues in existing promotion data integration component 

The issues This enhancement design addresses several issues in the existing Promotion data integration component in DWS. The issues are described in the following.

### Issue 1: manual process to add or change dimensional data in DWS 

   The existing system adopts an approach that is commonly used by developers in data warehouse system. Since the DWS developers are good at writing SQLs, they wrote the individual SQL insert/update/delete DML statments to add/change dimensional data in DWS dimensional data. This approach is simple and works great when the number of new or changed dimensional records is small (less than 20 records). 

   However, when the number of new or changed dimensional records is big (hundreds of records), this approach is heavy coding labored, non-visualized, hard to trace the DML statments in script.

   TODO: Diagram - load dimensional data using individual SQL insert/update/delete DML statments  

### Issue 2: Use visual ETL tool to process complex data transformation logic

   The DWS system uses Informatica jobs to extract, transform, and load Fact data. Visual ETL tools like Informatica, Talend work great when the data transformation logic is simple and straightforwd. 

   However, the visual ETL tools become unwieldly when the data transformation logic is complex.

   The visual ETL tools become unwieldly in complex data transformation logic. For a complex transformation logic, which could have been solved with several lines of Python or Java codes in clear logic flow, it will take many work-around steps in ETL tool to develped. The end result of using visual ETL tool to develop complex transformation logic is a visual flow chart that is highly complex, confusing, difficult to be undertood by other developers and thus difficult to maintain and enhance in the future. With the time going by, even the oroginal developers will have difficulty understanding the highly complex visual flow chart themselves.  
  
### Issue 3: Use visual ETL tool to process extra large of data traffic volumn (hundreds of millions of fact records) 

   With today's network speed, visual ETL tool like Informatica and Talend can handle a few millions of fact records with tolerable time performance. 
   
   However, when there are a few hundreds of millions of fact records, it will take hours to read the fact data from database, transform, and load them back to database. The time performance is untolerable. The issues lie in two areas: 1) reading and writing extra large number of fact records from/to database causes IO bandwidth bottleneck; 2) the large memory cache size and concurrent multi-threaded procesing on ETL server demands a machine with high CPU capacity and high RAM capacity. This will be costly in hardware investment. 
  
   For example, in the current promotion data integration system, it generates promotion data not only by monthly promotion turn, also by weekly, and by daily. The reason of generating weekly and daily promotion data is to provide a sales revenue weekly and daily drilldown view for BI report analysis. In each monthly promotion turn, there are avg 170k promotion items records, avg 467k promotion locations records. By multiplying by 4, there are 680k and 1,840k weekly records per month By multiplying by 30, there are 5,100k and 14,010k daily records per month. Assume each record avg 500 bytes, the size of promotion data generation can reach tens of billions of bytes in a fresh daily batch jobs run.

  
### Issue 4: hard coded codes/types mapping logic from upstream transactional system to DWS report system
   
   It was initially convenient to hard code codes/types mapping logic in ETL transformation when the number of mappings is small. With the upstream transactional system adds more and more codes/types, the hard coded mapping logic become a constant coding labor. Every time new promotion programs or new promotion locations are added in upstream transactional system, a hard coded mapping logic may have to be added. Hard coded mapping logic also makes the visual ETL flow chart bloated with branches of data process flows that only slightly differs in codes/types mapping logic. 
   
   For example, promotion data integation component maps promotion location codes used in transactional SAM system to the promotion subtypes used in DWS promotion report system. There are different codes/types mapping logics depending on the promotion types. 
    

# Enhancement design of Promotion data integration component in DWS

This session address issue 1 described in the above. 

## Automate new or changed dimensional data loading in DWS 

### Basic enhancement - Using business client CSV input files and PL/SQL Stored Procedures

When adding or chaning a large number of dimensional data records, The current approach is to write individual insert/update/delete SQL DML statements for each records to directly load into multiple inter-related final dimensional tables in DWS with surrogate IDs. Assume there are 50 of new or changed programs, and 5 final fact tables to populated, there will be, <number of new or changed programs> * <number of final dimensional tables>, total 250 SQL DML statments to hand written by developers.

The enhanced approach is to use business client provided CSV input files. The business client provides CSV input files containing new or changed business records, a **generic python script** is developed to read the business CSV input files, load the raw file contents to staging tables in DWS. PL/SQL stored procedures are developed to read dimensional records from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWS.

Assume there are 50 of new or changed programs, the enhanced approach will requires business client to provide a csv file with 50 lines of records.

We can see that using this business CSV input file approach, it still invoves developers' manual work to receive a new csv file from business client, verify the input file format, run the data staging job and PL/SQL jobs.

TODO: Diagram - load dimensional data using CSV input file + PL/SQL Stored Procedures approach


### Advanced enhancement - Using business client notifications, actions, submission and approval workflow

This advaned enhancement implements a fully automatic workflow that involves business client only. The approach will not involve the developers in the on-going dimensional data population, maintenance and governance. 

I have rarely seen this advaned enhancement in data warehouse systems. The rarity is mainly because the development of automatic workflow requires technical skills that go beyond DWS team normal skill sets. The workflow is actually an web application development. 

The high level design of the workflow is as the following:

1. developed a standalone web application that running forever. It is not a scheduled job.

2. the scheduled ETL batch jobs will extract the dimensional information from upstream data feeds, and populate the DWS dimensional tables with as-it-is data quality or even blank fields values which are necessary in DWS report system. If the upstream data feed introduces a new programs, the ETL jobs will add a new records to  dimensional table with many key field unpopulated.

3. the web app polls the dimensional tables in DWS a few times a day. If it detects any dimensional records with invalid field values or missing required field values, it will put the invalid dimensional records into a notification queue.

4. the web app sends email notification to the business client about the new or invalid dimensional records.

5. business client go to a browser to login to web app
	- the business client fills or corrects the dimensional records. submit and approve the changes. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "changed".
	- the business client can also query an existing valid dimensioal records, change some fields to new values, submit and approve the change. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "changed".
	- the business client cal also add a new dimensional records. submit and approve the new records. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "new". 

6. upon completion of daily new or changed dimensional records, the business client clicks a button to trigger the execution of the PL/SQL stored procedures, which will read the new or changed dimensional data from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWS.


TODO: Diagram - automatic dimensional data governance using notification and approval workflow.


## Use python to process complex data transformation logic

Python has powerful built-in core language features, and third party packages like Pandas and cx_Oracle, to process data extraction, transformation logic that can be as complex as you can image, and data uploading to dws.

when the data transformation logic is comlex, where visual ETL tool becomes unwieldy, I suggest writing a python script to process the transform logic.  Visual ETL tools like Informatica or Talend provides a UI command line component to invoke the python scripts.


## process extra large of data traffic volumn (hundreds of millions of fact records)  

I suggest using PL/SQL stored procedures to read/process/write extra larget of data transformation. This will avoid the IO network bottleneck, which is caused by the reading/writing extra large volumn of data from/to database to ETL application jobs. PL/SQL will keep reading and writing data local to the database server.

Avoid using cursors in PL/SQL SP. Stored Procedure codes writing with cursors can not be optimized by database execution plan. Instead, writing big SQLs and using temporary tables to process the large data transformation logic in PL/SQL SPs.

## Use mapping configuration tables instead of hard coded codes/types mapping logic

Data integration process always need to map codes/types from upstream transactional systems to DWS report system. Hard codes/types mapping is a no no, no matter how small number of the codes/types is. Always use a configuration table to map codes/types. 

For example, in promotion DI system, there is map logic to map from SAM system promotion location codes to DWS system promotion subtype codes. The sample hard coded mapping snippets are as the following. each similar mapping logic appars in two places, one place is in corporation items transformation phase , the other place is in corporation locations transformation phase.

This is a perfect scenario to move the hard coded mapping logic to a mapping configuration tables. 

We can see some mappings of codes/types are one-to-one straighforward. other mappings are based on certain field value patterns in a dimension record. We can use regualr expressions in the mapping configuration table. In the future, we only need to add or change the mapping configuration table records without touching the ETL codes. This will greatly simplfy the ETL code logic and improve the data integration component readibility, maintenability, and time to production.


Sample hard coded mapping snippets in existing ETL Informatica job:

For promotionType = 'EndAisle'

- during promotion items data integration phase, 

  1. first get end aisle position

			IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'P',1,1)-1))),   IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'A',1,1)-1))), 
			IIF( (INSTR(END_AISLE_NUMBER,'B',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'B',1,1)-1))), IIF(UPPER(LTRIM(RTRIM(END_AISLE_NUMBER)))='HD', 'HD',END_AISLE_NUMBER))))

  2. get end aisle flight	

			IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0 AND 
			INSTR(END_AISLE_NUMBER,'A',1) <>0 AND INSTR(END_AISLE_NUMBER,'B',1)<>0), 'PA+B', IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0), (SUBSTR(END_AISLE_NUMBER,(INSTR(END_AISLE_NUMBER,'P')))),
			IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0 AND INSTR(END_AISLE_NUMBER,'B',1)  <> 0 ), 'A+B',
			IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0), 'A',
			IIF( (INSTR(END_AISLE_NUMBER,'B',1) <>0), 'B',
			NULL)))))

  3. concate end aisle position and end aisle flight

  4. based on step 3 result, mapping the locations to subtypes:

			IIF(VALUE31 = 'HD', 'HD',
				 IIF(VALUE31 = 'GL', 'GL',
				 IIF(VALUE31 = '12P', 'EA12P',
				 IIF((INSTR(VALUE31,'FEM',1,1)) <> 0 , VALUE31,
				 IIF((INSTR(VALUE31,'PA+B',1,1)) <> 0,'EA' || LPAD(VALUE31,6,'0'),
				 IIF((INSTR(VALUE31,'PA',1,1)) <> 0, 'EA'||LPAD(VALUE31,4,'0'),
				  IIF((INSTR(VALUE31,'PB',1,1)) <> 0,'EA'||LPAD(VALUE31,4,'0'),
				  IIF((INSTR(VALUE31,'A+B',1,1)) <> 0, 'EA'||LPAD(VALUE31,5,'0'),
				  IIF((INSTR(VALUE31,'A',1,1)) <> 0, 'EA'||LPAD(VALUE31,3,'0'),
				  IIF((INSTR(VALUE31,'B',1,1)) <> 0, 'EA'||LPAD(VALUE31,3,'0'),
				  IIF((INSTR(VALUE31,'VL',1,1)) <> 0, 'EA'||VALUE31,
			IIF((INSTR(VALUE31,'CE',1,1)) = 1, VALUE31,
			IIF((INSTR(VALUE31,'EZ',1,1)) =1, VALUE31,
			IIF((INSTR(VALUE31,'S',1,1)) =1, VALUE31,
			IIF((INSTR(VALUE31,'W',1,1)) =1, VALUE31,
			'EA'||LPAD(VALUE31,2,'0'))))))))))))))))

- during the promotion locations data integration phase:

			DECODE(TRUE, 
			INSTR(LOCATION_CODE31,'P') <> 0, lpad(LOCATION_CODE31,3,'0'),
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'GL')  = 1, LOCATION_CODE31,
			LTRIM(RTRIM(UPPER(LOCATION_NAME31))) = 'GO LOCAL','GL',
			UPPER(LTRIM(RTRIM(LOCATION_CODE31))) = 'HERO','HD',
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'VL') = 1, 'EA'||LOCATION_CODE31,
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'FEM') > 0, LOCATION_CODE31, 
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'CE')  = 1, LOCATION_CODE31,
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'EZ')  = 1, LOCATION_CODE31,
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'S')  = 1, LOCATION_CODE31,
			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'W') = 1, LOCATION_CODE31,
			'EA'||lpad(LOCATION_CODE31,2,'0'))

			DECODE(TRUE,
				INSTR(LOCATION_CODE34,'P') <> 0, 'EA'||lpad(LOCATION_CODE34,3,'0')||'B', IN (LOCATION_CODE34 ,'4','8','9','12',0) = 1, 'EA'||lpad(LOCATION_CODE34,2,'0')||'B',
				'EA'||lpad(LOCATION_CODE34,2,'0'))

			DECODE(TRUE,
			INSTR(LOCATION_CODE5,'P') <> 0, 'EA'||lpad(LOCATION_CODE5,3,'0')||'A+B',
			'EA'||lpad(LOCATION_CODE5,2,'0')||'A+B')

For promotionType = 'ProductExtender', then

	DECODE(UPPER(LTRIM(RTRIM(DESCRIPTION))),
	       'SHELF EXTENDERS - COMMUNITY', 'PECM',
	       'SHELF EXTENDERS - GREEN','PEGR',
		   'SHELF EXTENDERS - REGULAR', 'PERG', 
		   'DISCOVERY','PEDI')

For promotionType = 'AirMiles', then 	

		IIF(ACRONYM34='BAM', 'AMRG',IIF(ACRONYM34='BBAM', 'AMBB','AMSB'))
		
For promotionType = "LimitedSales", then

		IIF(VALUE39 = 'Super Sale LTO', 'LS',
		IIF(VALUE39 = 'Flash Sale', 'LF', NULL))
		
For promotionType = 'Mini Thematic', then

		'MI'||'-'||substr(VALUE38, -1, 1)	
		
	

## Technology Stack used in automation enhancements
PL/SQL Stored Procedures   
Python    
Informatica  
Java, JEE, Spring-web  


# Code Snippets



# Deployment
Database server  
Informatica server  
Application server  


# Conclusion  
The design ideas described in this document is applicable when the team skill sets and circumstances is suitable.

In practical, designs always need to be adapted to a team strong skill sets. The team will be the one supporting and maintaining the data integration and data warehouse system on on-going basis.








