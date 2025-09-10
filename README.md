**Football Data Analysis Project**

**Project Overview**
This project demonstrates the complete process of football data analysis, from raw CSV files to interactive dashboards using Excel, SQL Server and Power BI. Each step is documented with clear explanations and visualizations to ensure transparency and reproducibility.

**Data Size and Processing Approach**
The dataset includes a combination of large files that exceed Excel’s capacity and smaller files that comfortably fit within Excel and Power Query limits.

Large datasets are processed directly in SQL Server to leverage its high performance, scalability, and robust data integrity features. SQL Server handles complex transformations, data validation, and aggregation efficiently, making it ideal for heavy data processing tasks.

Smaller datasets (approximately 1000 records) are processed in Power Query. Power Query offers a flexible, user-friendly environment for data cleaning, filling missing values, renaming columns, and performing lightweight transformations without requiring advanced SQL skills or database access.

This hybrid approach optimizes the workflow by balancing performance and ease of use, ensuring that each data subset is handled in the most appropriate environment.

**Data Backup Procedure**

**Overview**

As a best practice and to ensure data integrity, a full backup of all source data files was performed prior to any data processing or transformation. This step guarantees that the original data remains intact and recoverable in case of any issues during the cleaning or analysis phases.

**Backup Details**
Source folder: C:\football data analysis\sources
Backup folder: C:\football data analysis\backup
Backup method:
All files from the source folder were copied to the backup folder.
Each backup file was renamed by appending the suffix _backup before the file extension to clearly distinguish backup copies from original files.
The backup folder was created if it did not already exist.
Backup Script (PowerShell)
The backup was automated using the following PowerShell script:

**PowerShell**
```
$sourceFolder = "C:\football data analysis\sources"
$backupFolder = "C:\football data analysis\backup"

if (-not (Test-Path -Path $backupFolder)) {
    New-Item -ItemType Directory -Path $backupFolder
}

Get-ChildItem -Path $sourceFolder -File | ForEach-Object {
    $originalName = $_.BaseName
    $extension = $_.Extension
    $newName = "${originalName}_backup${extension}"
    $destinationPath = Join-Path -Path $backupFolder -ChildPath $newName

    Copy-Item -Path $_.FullName -Destination $destinationPath -Force

    Write-Host "Copied: $($_.Name) -> $newName"
}
```
This backup step is a critical part of the data preparation workflow and reflects a professional approach to data management.

**Preliminary Data Model Diagram**

This document presents the initial version of the data model diagram, which provides a comprehensive overview of how the datasets are interconnected.

This preliminary diagram serves several key purposes:

It allows for a clear understanding of the relationships and dependencies between different data tables.
It facilitates the identification of potential improvements, such as normalization opportunities, redundant data removal, or necessary schema adjustments.
It acts as a foundational blueprint to guide further development, refinement, and optimization of the data model.
By reviewing this diagram, stakeholders and developers can collaboratively assess which elements of the model should be modified, retained, or enhanced to ensure data integrity, performance, and scalability.

[FootballAnalysisver1.pdf](https://github.com/user-attachments/files/22115818/FootballAnalysisver1.pdf)

**Removal of URL Fields from Source Data**
**Overview**
As part of the data preparation and cleaning process, all URL fields present in the source datasets have been removed. This decision was made because URLs are not required for the subsequent stages of data analysis and modeling.

Rationale
URLs do not contribute to analytical insights such as player performance, match outcomes, or transfer values.
Removing URLs reduces data volume and complexity, improving processing speed and storage efficiency.
It helps maintain focus on relevant data attributes, ensuring cleaner and more manageable datasets.
Eliminates potential privacy or security concerns related to external links.
Implementation
During the Power Query cleaning phase, all columns containing URLs (e.g., url, image_url) were identified and removed from the datasets.
This step was applied consistently across all relevant tables to maintain uniformity.


**Creation of the managers Dimension Table**
Overview
The managers dimension table was created by extracting a unique list of manager names from the games and club_games fact tables. To transform this simple lookup list into a robust and analytically flexible dimension, the name attribute was further processed.

Advanced Name Parsing for Enhanced Analytics
A single FullName column limits analytical capabilities such as sorting by surname. To overcome this, the FullName field was deconstructed into atomic components, adhering to best data modeling practices.

The following columns now constitute the managers dimension:

**Manager_ID:** A unique, integer-based primary key generated to ensure efficient relationships.
**FirstName:** Contains the manager's first name.
**LastName:** Contains only the final part of the manager's name.
Logic: To create a consistent sorting key, only the text after the last space was extracted. This effectively handles multi-part surnames (e.g., for "Erik ten Hag", the LastName is "Hag"), providing a clean and reliable field for alphabetical sorting.
**FullName:** The original, cleaned full name was retained for straightforward display purposes in reports and visuals.
This structured approach elevates the managers table from a simple lookup list to a professional dimension, enabling more sophisticated sorting, filtering, and analysis while maintaining a user-friendly display name.

**competitions Table** - Standardization and Refactoring

The competitions dimension plays a fundamental role in data segmentation. The transformation process focused on converting raw, technical data into clean and consistent categorical attributes, ready for use in the presentation layer.

1. Text Cleaning and Standardization

Categorical Attributes (name, sub_type, type, confederation): All text-based attributes were standardized. Operations included removing special characters (hyphens, underscores) and applying Proper Case formatting. This ensures a consistent and professional appearance for labels in filters and charts.
2. Data Type Correction

Logical Conversion: The is_major_national_league column, which stored boolean information as text, was converted to a native True/False data type. This transformation is key to ensuring data integrity and enabling efficient logical operations in future analyses.
3. Normalization and Integration

Redundancy Removal: The competition_code column was identified as redundant, as its values were nearly identical to the data in the name column. It was removed to optimize and simplify the data schema.
Integration with countries Dimension: The original text-based country_name column was replaced with a country_id foreign key. This establishes a formal, integer-based relationship with the newly created countries dimension, ensuring that all geographical information is managed centrally and efficiently.
As a result of these actions, the competitions table has become a fully normalized and clean dimension, providing a solid foundation for its relationships with fact tables.
**Data Normalization and Advanced ETL Challenges**
Overview of Data Normalization Strategy
A core objective of the data preparation phase was to transform the raw, denormalized dataset into a robust and efficient star schema. This involved creating dedicated dimension tables for key business entities like countries, competitions, and players. This professional approach ensures data integrity, improves query performance, and enhances the overall scalability of the model.

This section details the creation of the geographies dimension and the subsequent refactoring of the competitions table, including the advanced challenges encountered and the solutions implemented.

1. Engineering the geographies Dimension
Initial Challenge: Handling Countries and International Competitions

The raw data lacked a central dimension for geographical entities. Country names were stored as text fields across players and competitions, while international tournaments were ambiguously marked with null values or a -1 identifier in the country_id column. A simple countries dimension was insufficient, as it would fail to uniquely identify non-domestic competitions like the "UEFA Champions League".

Solution: Creation of a Composite geographies Dimension

A more sophisticated dimension named geographies was engineered to serve as a single source of truth for both countries and unique international competitions.

Implementation Steps:

Data Aggregation: A master list was created by appending distinct country names from the players and competitions tables along with the unique names of international competitions (identified by null country names).
Standardization: The master list was thoroughly cleaned using Power Query transformations, including trimming whitespace, removing non-printable characters, standardizing case (Proper Case), and replacing delimiters (-, _).
Indexing: A unique, integer-based primary key (geo_id) was generated to ensure efficient, error-free relationships.
This process resulted in a robust dimension table capable of uniquely identifying every geographical or competitive entity in the dataset.

2. Refactoring the competitions Table
Advanced Challenge: Resolving a Cyclic Reference Error

The primary challenge during the transformation of the competitions table was a cyclic reference error. This critical issue arose because:

The geographies query depended on the competitions table to source its list of entities.
Simultaneously, the competitions query attempted to merge with the geographies table to retrieve the geo_id.
This created an unbreakable dependency loop (geographies -> competitions -> geographies), causing Power Query's evaluation engine to fail. This error manifested as unresponsive previews, failed merges, and misleading error messages until the root cause was identified.

Solution: Decoupling Dependencies with Referenced Queries

The cyclic dependency was resolved using a standard data warehousing best practice: decoupling queries via references.

Implementation Steps:

Buffered References: "Snapshot" references of the source queries (competitions and players) were created, named ref_competitions and ref_players. These references act as stable, intermediate buffers that do not change when the original queries are modified.
Redirecting Dependencies: The geographies query was modified to source its data exclusively from these non-dependent references (ref_competitions, ref_players), thus breaking the cycle.
Successful Merge: With the dependency loop eliminated, the competitions table could be safely merged with the now-independent geographies dimension.
Final Refactoring of competitions:

A conditional lookup_key column was engineered to handle both country names and international competition names, ensuring a 100% match rate against the geographies dimension.
The merge operation was used to join competitions with geographies on this key, successfully adding the geography_id foreign key.
All redundant and intermediate columns (country_name, old country_id, lookup_key) were removed, and the remaining columns were cleaned and standardized.

**Data Quality, Standardization, and Enrichment for the clubs Dimension**
The clubs dimension serves as a central reference for all club-related attributes. To elevate it to a professional standard, the table underwent a comprehensive transformation process focusing on data cleansing, normalization, handling of inconsistent data, and feature engineering for enhanced analytical value.

1. Removal of Incomplete and Redundant Columns

100% Empty Columns: The total_market_value and coach_name columns were removed from the table as they were found to contain no data (100% null values). This step simplifies the model and eliminates informational "noise".
Rationale: Retaining entirely empty columns provides no analytical value and is contrary to clean data modeling principles. The correct historical relationship between clubs and coaches is managed in the club_games fact table.
2. Data Cleaning and Type Correction

Textual Attributes: All descriptive text fields, such as name and stadium_name, were standardized using a consistent cleaning process (Trim, Clean, Proper Case).
Numerical Attributes:
The average_age column, originally stored as text with dots as decimal separators, was correctly converted to the Decimal Number data type using locale-aware transformations.
Other numeric columns (squad_size, stadium_seats, etc.) were set to the Whole Number type.
3. Advanced Handling of Financial and Statistical Data

Parsing Financial Text: The net_transfer_record column, stored in a complex text format (e.g., +€778m, +-0), was systematically parsed into a clean Decimal Number. This involved:
Handling special cases like +-0.
Creating a conditional Multiplier column to handle suffixes ('m' for millions, 'k' for thousands).
Removing non-numeric characters (€, letters).
Calculating the final numeric value.
Contextual Handling of Zeros and Nulls: A deliberate, context-aware strategy was applied to zero values to ensure analytical integrity:
In squad_size and foreigners_number, a value of 0 was treated as valid data (e.g., "zero players") and was preserved.
In average_age and foreigners_percentage, a value of 0 was identified as logically impossible or misleading. These zeros were replaced with null to prevent them from skewing aggregate calculations like averages.
4. Feature Engineering for Deeper Insights

Stadium Size Category: To facilitate cohort analysis, a new categorical column was engineered based on the stadium_seats attribute. Clubs were grouped into "Small" (< 15k seats), "Medium" (15k-40k), and "Large" (> 40k) categories. This enables powerful comparisons of club performance based on stadium size.
5. Schema Standardization and Finalization

Key Renaming Convention: To ensure model-wide consistency, primary and foreign keys were renamed using a "Key" suffix. club_id was renamed to ClubKey, and domestic_competition_id was renamed to CompetitionKey.
Final Review: All column names were standardized to a clear, English-based convention, and their final order was arranged logically for improved readability.
These comprehensive refinements have transformed the clubs table into a robust, clean, and analytically enriched dimension, fully prepared to support complex queries and deliver reliable insights within the data model.

**Normalization, Cleansing, and Enrichment of the Core players Dimension**
The players table is the most critical and attribute-rich dimension in the data model, serving as the foundation for all player-centric analysis. It underwent an extensive transformation process focused on aggressive normalization, data cleansing, and sophisticated feature engineering to convert it into a lean, robust, and analytically powerful asset.

1. Aggressive Normalization
To adhere to the principles of a star schema and ensure a single source of truth, all attributes describing a player's current club were removed.

Removed Columns: current_club_name, current_club_domestic_competition_id.
Rationale: This information is now accessed exclusively through a relationship between players[ClubKey] and clubs[ClubKey]. This critical step prevents data redundancy, eliminates the risk of inconsistencies, and significantly optimizes the data model.
2. Data Pruning and Optimization

Unnecessary Columns: The agent_name and contract_expiration_date columns were removed as they were identified as not required for the analytical objectives of this project.
Technical Identifiers: The player_code column, a non-analytical identifier, was also removed.
Rationale: This pruning process simplifies the model, reduces its memory footprint, and focuses the dimension on the most relevant attributes for analysis.
3. Advanced Feature Engineering for Deeper Insights
Several new, high-value attributes were engineered to enrich the dimension:

Atomic Name Columns (FirstName, LastName): The composite FullName attribute was deconstructed into separate FirstName and LastName columns. The LastName was defined as the text after the last space to ensure consistent sorting for multi-part names.
Standardized Season Format (last_season): The numeric year (e.g., 2023) was transformed into a clear, standardized text format (2022/2023) for improved readability and unambiguous interpretation in reports.
Calculated Age: A player's current age was calculated based on their date_of_birth. A robust, error-proof M formula (try...otherwise) using the stable DateTime.FixedLocalNow() function was implemented to handle potential data quality issues in the source column.
Career Stage Category: A conditional column was created to segment players into analytical cohorts ("Young Talent", "Peak Career", "Veteran") based on their calculated age.
4. Data Type Correction and Cleansing

Date Conversion: The date_of_birth column was converted from a DateTime type (containing a redundant 00:00:00 time component) to a clean Date type. A locale-aware transformation was used to correctly parse the DD.MM.RRRR source format.
Handling of Missing Categorical Data (foot): Null values in the foot column were replaced with the descriptive text "Unknown" to create an explicit, filterable category for end-users.
5. Strategic Denormalization for Performance

After careful consideration, the market_value_in_eur and highest_market_value_in_eur columns were intentionally retained in the dimension table.
Rationale: While this information also exists transactionally in player_valuations, keeping these key performance indicators directly in the players dimension is a deliberate denormalization strategy. It significantly boosts report performance for common operations like sorting and filtering by player value, and simplifies the user experience by providing ready-to-use measures.


**Overview: A Hybrid Approach to Time Intelligence**
To enable sophisticated time-based analysis, a dedicated Calendar Dimension (dim_Calendar) was created. This is a fundamental best practice in business intelligence, as it provides a stable and attribute-rich foundation for all time intelligence functions.

A hybrid approach was adopted to construct this dimension, leveraging the strengths of both SQL Server and Power Query:

SQL Server was used for its high-performance, set-based processing capabilities to generate the raw, continuous sequence of dates.
Power Query was then used to integrate this calendar into the existing data model and prepare the fact tables for their relationship with this new dimension.
Part 1: High-Performance Date Generation in SQL Server
The core of the calendar table—a continuous sequence of dates spanning the entire scope of the project's data—was generated using a T-SQL script in SQL Server.

Implementation Steps in SQL:

Static Date Range Definition: To ensure stability and accommodate future data, a fixed date range was defined (e.g., from 1993-07-01 to 2026-07-01). This approach was chosen over dynamic range finding to create a robust and predictable time axis.
Table Structure Creation: A new table, dbo.dim_Calendar, was created with a predefined schema. This included a primary key (DateKey in YYYYMMDD integer format), a FullDate column, and numerous attribute columns (e.g., YearNum, MonthName, DayOfWeek, QuarterName).
Iterative Row Generation: A WHILE loop was used to iterate from the defined start date to the end date, inserting one row for each day into the dim_Calendar table. During the INSERT operation, both the FullDate and the DateKey primary key were populated simultaneously to ensure data integrity.
Attribute Calculation: After the primary data generation, a single, efficient UPDATE statement was executed to calculate and populate all descriptive attributes for every date in the table (e.g., deriving MonthName from FullDate).
Rationale for Using SQL Server:

Performance: T-SQL is exceptionally efficient at generating large, set-based data like a multi-decade calendar.
Scalability: The resulting table is a persistent, reusable asset in the database, available to any other analytical tool.
Robustness: The scripted approach is fully documented, repeatable, and independent of the primary data transformation workflow in Power Query.
Part 2: Integration and Model Refactoring in Power Query
Once the dim_Calendar table was created in SQL Server, it was integrated into the Excel data model via Power Query. The primary task was to refactor the fact tables to use the new calendar dimension.

Implementation Steps in Power Query:

Import Calendar Dimension: A new query was created to connect to the SQL Server database and import the dbo.dim_Calendar table into the Power Query Editor. It was renamed to Calendar.

Refactoring Fact Tables: Each fact table containing a date column (e.g., games, transfers, player_valuations) underwent a transformation to replace its native date column with the corresponding DateKey from the Calendar dimension. This was achieved through the following sub-process for each fact table:

Merge Operation: The fact table was merged (Left Outer Join) with the Calendar query, using the native date column (e.g., games[date]) and Calendar[FullDate] as the join keys.
Key Expansion: The resulting nested table column was expanded to bring only the DateKey into the fact table.
Schema Cleansing: The original, now-redundant native date column was removed. The new DateKey column was positioned with other keys.
Final Outcome:

The data model now contains a single, authoritative Calendar dimension.
All fact tables have been successfully refactored to include a DateKey foreign key, replacing their disparate date columns.
This clean, star-schema structure, with a dedicated Calendar dimension, fully enables advanced time intelligence calculations and provides a consistent, user-friendly experience for filtering and analyzing data over time. The final relationships between the Calendar and the fact tables are established in the Power Pivot data model.




