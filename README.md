# Power-BI-Assignment-1-Data-Transformation-Data-Modeling
# Power-BI-Assignment-1
## Assignment-1-Data-Transformation-and-Data-Modeling

* **Problem Description:** Analysed an e-commerce sales dataset made up of three tables (List of Orders, Order Details and Sales Target) to understand orders, profit and monthly sales targets. The work covers importing the data, cleaning and transforming it in Power Query, merging the tables, handling missing and duplicate data, sorting, filtering, grouping, and building a data model with relationships.
* **Tools Used:** Power BI Desktop, Power Query Editor (M language) and the Model view.

### Source Tables

* **List of Orders:** 560 records (500 with data, 60 empty). Columns: Order ID, Order Date, CustomerName, State, City.
* **Order Details:** 1,500 line items. Columns: Order ID, Amount, Profit, Quantity, Category, Sub-Category.
* **Sales Target:** 36 rows (12 months x 3 categories). Columns: Month of Order Date, Category, Target.

## 1. Data Import

* **Importing Data:** Loaded the three CSV files using *Get Data > Text/CSV > Transform Data*.
  1. Loaded `List of Orders.csv` and opened it in Power Query Editor.
  2. Added `Order Details.csv` and `Sales target.csv` using *New Source > Text/CSV*.

## 2. Data Transformation

### List of Orders

* **Restricting Rows:** Kept only the first 500 rows using *Keep Top Rows*. The last 60 rows of the file are completely empty.
* **Order Date:** Changed `Order Date` to the Date type using *Change Type > Using Locale* with English (United Kingdom), because dates are in `DD-MM-YYYY` format and would otherwise be misread.
* **Proper Case:** Formatted `CustomerName` using *Format > Capitalize Each Word* (`Text.Proper`) for consistent capitalisation.
* **Location Column:** Created `Location` in the format City, State with the custom column formula `[City] & ", " & [State]`. The original `City` and `State` columns were kept.

### Order Details

* **Fixed Decimal Number:** Changed the data type of `Amount` to Fixed Decimal Number.
* **Profit Margin:** Created a custom column using `[Profit] / [Amount]` and formatted it as a *percentage*. A zero `Amount` returns null to avoid a divide-by-zero error. It is created here because `Profit` and `Amount` exist only in this table.
* **Profit Status:** Added a conditional column using nested `if`:
  1. Profit below 0 = Loss
  2. Profit equal to 0 = Break-Even
  3. Profit above 0 = Profit

  Result: 950 Profit, 503 Loss and 47 Break-Even line items.

### Sales Target

* **Fixed Decimal Number:** Changed the data type of `Target` to Fixed Decimal Number.

## 3. Merging Data (Joins)

* **Orders Data:** Merged `List of Orders` and `Order Details` using *Merge Queries as New* on `Order ID`.
  1. Join kind: **Left Outer**, so every retained order is kept.
  2. ~~Inner join~~ was not used, because it would silently drop any order without details.
  3. Result: 1,500 rows x 13 columns. One row is one line item, because an order can contain several products.

## 4. Handling Missing Data and Duplicate Data

### Missing Values

* **Finding:** Checked every table using Column Quality and Column Profile. Missing values were found only in List of Orders (60 trailing empty rows). Order Details and Sales Target have 0 missing cells.
* **Strategy:** Removed the empty rows by keeping the first 500 rows. Financial values (`Amount`, `Profit`, `Target`) were not imputed, and the query raises an error if a required value is ever null.

### Duplicates

* **Orders retained:** 500 rows with 500 distinct `Order ID` values and no duplicates. Strategy: remove exact duplicates and reject conflicting records for the same `Order ID`.
* **Order Details:** 0 exact duplicates. ~~Removing duplicate Order IDs~~ was not done, because repeated Order IDs are valid line items.
* **Sales Target:** 0 exact duplicates. Repeated `Category` values were preserved because targets are monthly.

## 5. Sorting and Filtering

* **Sorting:** Sorted `Orders Data` by `Order Date` in descending order so the newest orders appear first.
* **Filtering:** Filtered `State` to Tamil Nadu for regional analysis, which gives 25 rows.
* **Orders Data All:** Retains the full merged data. Both queries sort newest first.

## 6. Grouping and Aggregating

* **Count of Each Order ID:** Duplicated `Order Details` and used *Group By* with Count Rows on `Order ID`. This gives 500 orders, of which 235 have a single line item and the largest have 12.
* **Average Profit by Category:** Used *Group By* on `Category` with Average of `Profit`.
  1. Electronics: 34.07
  2. Clothing: 11.76
  3. Furniture: 9.46
* **Total Amount by Sub-Category:** Used *Group By* on `Sub-Category` with Sum of `Amount`. The top five are Printers (58,252), Bookcases (56,861), Saree (53,511), Phones (46,119) and Electronic Games (39,168).
* **Total Target by Month:** Duplicated `Sales Target` and used *Group By* on `Month of Order Date` with Sum of `Target`. The text month (for example Apr-18) was converted to a date so months sort in calendar order.

## 7. Data Modeling

* **Relationship 1 (Order ID):** `List of Orders[Order ID]` to `Order Details[Order ID]`, One-to-Many (1:*). Active, single direction from List of Orders to Order Details.
* **Relationship 2 (Category):** `Order Details[Category]` to `Sales Target[Category]`, Many-to-Many (*:*), because Category repeats in both tables. Active, checked in *Manage Relationships*.
* **Model Limitation:** The Category relationship does not match on month or state. Targets are national monthly values, so they should not be interpreted as regional targets.
