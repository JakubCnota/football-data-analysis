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

