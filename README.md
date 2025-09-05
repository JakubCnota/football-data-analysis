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

In the raw dataset, manager information (full names) was stored as text fields directly within fact tables such as games and club_games. This structure led to massive data redundancy, increased the risk of inconsistencies (e.g., due to typos), and negatively impacted the model's performance.

To normalize the data model and ensure data integrity and efficiency, a decision was made to create a dedicated managers dimension table.

Implementation Process

The process of creating the managers table and refactoring the model was carried out in the following steps:

Data Aggregation: Data from the games and club_games tables were combined to gather all occurrences of manager names that appeared in the dataset.
Extraction of Unique Values: A unique list of managers was extracted from the aggregated set, eliminating all duplicates.
Creation of a Primary Key (Index): An index column (manager_id) was added to the unique list, assigning a unique numeric identifier to each manager. This identifier became the primary key of the new table.
Creation of the Dimension Table: Based on this list, a new, clean dimension table – managers – was created, containing the manager_id and full_name.
Updating Fact Tables: The manager names in the original tables (club_games and games) were replaced with their corresponding manager_id values. These columns became foreign keys, establishing a formal relationship with the new managers table.
Rationale and Benefits

This process yielded four key benefits:

Data Integrity: A manager's name is now stored in a single place. Any corrections or updates only need to be made once in the managers table, and the changes will be automatically reflected throughout the entire report.
Improved Performance: Filtering and aggregating data using numeric (integer) keys is significantly faster than performing operations on text fields.
Reduced Model Size: Storing integers instead of repetitive, long text strings reduces memory consumption.
Model Clarity: An explicit and logical one-to-many relationship was established between the dimension (managers) and the facts (club_games), adhering to the best practices of star schema modeling.
