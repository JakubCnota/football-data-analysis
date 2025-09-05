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




