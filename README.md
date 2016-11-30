
# udd-validator-command-line-tools
This contains a series of command line tools for validating UDD data and related tasks to make life easier. All written in C# using .NET v4.6.1

The main command line tool is the Single Json File Validator (release version) or alternatively the Dynamics Json Test App for those who want debugging capabilities.

# Single Json File Validator (SingleJsonFileValidator.exe)
This command line tool works in the following manner:
1. Go fetch the UDD Specifications as a list of "Components"
2. Load the data from either a JSON or XML file into a global collection
3. Validate the date
4. Create an Excel report

The shared components that the Single Json File Validator uses are the following:
UDD Model Extractor
UDD Data Loader
UDD Data Validator
Excel Report Generator Without Excel

More details about each share component can be seen below.


# UDD Model Extractor
This fetches a version specific UDD specification found in the root folder of https://github.com/jiscdev/analytics-udd/tree/ and creates a collection of "Components" each mapping to a specific entity e.g. student or course. Each entity is then fleshed and a new fields collection is created under each entity.

Each "Component" has the following properties:
Version
Name
File Name
Fields -> holding a collection of "Field" objects each with a Name, Is Mandatory, Field Type Name (e.g. string, int, datetime etc), Field Type Precision (i.e. length of string or precision of double/float), Foreign Key Definition, a list of Valid Values
Relationships -> holding a collection of "Relationship" objects each with a Name and a Related Object Name which allows us to go from one component to another easily
Primary Key Definition -> holding up to 3 fields which make up the primary or composite primary key


# UDD Data Loader
The purpose of the UDD Data Loader is to match a physical file holding either JSON or XML data, load all of the data into a global static array and map the data quickly to the correct component found from the UDD Model Extractor. Data held in a collection is much easier to both manage and quicker to read in the validation process than trundling through each line in a file. A global static collection is created for each Component found from the UDD Model Extractor.


# UDD Data Validator
Based on the list of Components extracted used the UDD Model Extractor, and the list of data items extracted in the UDD Data Loader, the purpose of this tool is to go through each Component, get the related Json objects (held as dynamic C# objects) as "records", then validate each "record" and produce a report at the end.

Each record goes through the following validation checks:
1. Does the record include fields that are different to those found in the UDD specification Component? (excluding those deprecated fields)
2. Does the UDD specification Component include fields which are not found in the record, which are mandatory and wich are not defined as a deprecated field?

Each field under each record goes through the following validation checks:
3. Can the field's value be parsed into the expected type (found by the Field Type Name)?
4. Does the field's value match the precision/length defined by the field's Field Type Precision value?
5. Is the field mandatory and is there a value?
6. If the field has a list of "Valid Values", does the field's value hold a valid value?

Each component goes through the aditional following validation checks:
7. Test each record's primary key or composite primary key values against all others in the global collection.

Now, the validator runs the full validation process using a series of Parallel.ForEach calls, holding collections in global static ConcurrentBag<T> properties. The result is a fairly quick turnaround. A JSON file holding 481,000 records, each with 16 fields, equates to 7,696,000 fields to check. 6 checks each making 46,176,000 checks... and this is all done in about 36 seconds on a 5 year old twin Xeon server. 

Within the final report produced, a series of groupings are made for records failing the same checks.


# Excel Report Generator Without Excel
Using EPPlus, and the report created using the UDD Data Validator, this tool creates a Jisc templated workbook detailing what was found in the validation process. For each error grouping, an additional worksheet is created within the same workbook such that users can easily find out what's wrong with each record. 


# Additional Tools
Additional tools have been created to make life easier. These will be explained in due course.


# Open Source Credits
There are a few notable mentions to make which have made things a LOT easier. 

NewtonSoft.Json - for the ability to make the reading and writing of Json files considerably simpler. More information can be found here: http://www.newtonsoft.com/json

EPPlus - for an ability to create Excel workbooks and worksheets without having the Microsoft Excel installed. More information can be found here: 
http://epplus.codeplex.com/


