# Power BI Assignment 1 – Data Transformation and Data Modeling

## Problem Description

Worked with an **e-commerce sales dataset** made up of three tables: *List of Orders*, *Order Details* and *Sales Target*. The goal was to prepare the data for analysis in Power BI.

The work covers:

* Importing the data
* Cleaning and transforming it in Power Query
* Merging the tables
* Handling missing and duplicate data
* Sorting, filtering and grouping
* Building a data model with relationships

## Dataset Overview

### List of Orders

* 560 rows in the file, but rows 501 to 560 are completely blank
* Columns: *Order ID*, *Order Date*, *CustomerName*, *State*, *City*
* *Order Date* is stored as text in DD-MM-YYYY format

### Order Details

* 1,500 rows, with no missing values and no duplicate rows
* 500 unique *Order ID* values, repeated because each row is one line item
* Columns: *Order ID*, *Amount*, *Profit*, *Quantity*, *Category*, *Sub-Category*

### Sales Target

* 36 rows: 3 categories for 12 months each
* *Month of Order Date* is stored as text, for example Apr-18

## Data Transformation

### Importing Data

1. Loaded **List of Orders.csv** using Get Data > Text/CSV and opened it with Transform Data
2. Added **Order Details.csv** and **Sales target.csv** in the same editor using New Source > Text/CSV

### Restricting Rows

* Kept only the first 500 rows of *List of Orders* using Keep Top Rows
* This also removes the empty rows at the end of the file

### Data Types

* **Order Date:** changed to Date using Change Type > Using Locale with English (United Kingdom), because the dates are in DD-MM-YYYY format
* **Amount** and **Target:** changed to Fixed Decimal Number

### Text Formatting

* Formatted *CustomerName* using Format > Capitalize Each Word so every name has consistent capitalisation
* Trimmed extra spaces in *State* and *City*, which fixed a value stored as "Kerala " with a trailing space

### New Columns

1. **Location:** custom column using *City* and *State* in the format City, State. The original columns were kept
2. **Profit Margin:** custom column in *Order Details* that divides *Profit* by *Amount* and is formatted as a percentage. A null check avoids a divide-by-zero error
3. **Profit Status:** conditional column with three labels
  1. Profit below 0 = **Loss**
  2. Profit equal to 0 = **Break-Even**
  3. Profit above 0 = **Profit**

Result of the Profit Status column: 503 Loss, 47 Break-Even and 950 Profit rows.

## Merging Data

* Merged *List of Orders* and *Order Details* into a new table named **Orders Data** using Merge Queries as New on the *Order ID* column
* Used a **Left Outer** join so every order is kept, then expanded the details columns
* Result: 1,500 rows, and every order found a match

## Missing and Duplicate Data

### Handling Missing Values

* Turned on Column Quality and Column Profile and checked the empty percentage of every column in all three tables
* Found 60 blank rows at the end of *List of Orders* and removed them with Remove Blank Rows
* ~~Imputing missing financial values~~ was not done: values such as *Amount*, *Profit* and *Target* are not filled in with guesses

### Handling Duplicates

* Checked *Order ID* in *List of Orders* and removed duplicates with Remove Duplicates
* **Kept** repeated *Order ID* values in *Order Details*, because each record is a separate line item
* **Kept** repeated *Category* values in *Sales Target*, because targets are monthly

## Sorting and Filtering

* **Sorting:** created a reference of *Orders Data* and sorted *Order Date* in descending order so the newest orders appear first
* **Filtering:** filtered *State* to Tamil Nadu for regional analysis, which gives 25 line items
* Kept **Orders Data All** as the full merged data

## Grouping and Aggregating

1. **Count of each Order ID:** duplicated *Order Details* and used Group By on *Order ID* with Count Rows
2. **Average profit by Category:** duplicated *Order Details* and used Group By on *Category* with Average on *Profit*
3. **Total amount by Sub-Category:** duplicated *Order Details* and used Group By on *Sub-Category* with Sum on *Amount*
4. **Total target by Month:** duplicated *Sales Target* and used Group By on *Month of Order Date* with Sum on *Target*

Converted the text month (for example Apr-18) to a date so the months sort in calendar order.

Check values for average profit: Clothing 11.76, Electronics 34.07, Furniture 9.46.

## Data Modeling

### Relationship 1 – Order ID

* Connected *List of Orders[Order ID]* to *Order Details[Order ID]*
* Type: **One-to-Many (1:\*)**, single direction, active

### Relationship 2 – Category

* Connected *Order Details[Category]* to *Sales Target[Category]*
* Type: **Many-to-Many (\*:\*)**, because Category repeats in both tables
* Confirmed in Manage Relationships that it is active

### Model Limitation

* The Category relationship does not match on month or state
* The targets are *national monthly values* and should not be read as regional targets

## Screenshots to Add

1. Power Query Editor showing the applied steps for *List of Orders*
2. Column Quality and Column Profile for the missing-value check
3. The *Orders Data* merge result
4. The grouped tables
5. Model view with both relationships

## Source Snapshots

* The three CSV files are embedded in the queries, so the report refreshes without a Downloads folder
* The original CSV files are also included in this repository
