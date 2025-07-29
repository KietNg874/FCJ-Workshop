---
title : "Lambda Functions"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Lambda functions are the core processing components of our serverless backup solution. In this section, you'll create three specialized functions that handle different aspects of the backup and recovery process.

![Lambda Functions Overview](/FCJ-Workshop/images/3.svlessimp/001-overview.png)

## Function Architecture Overview

Our serverless backup solution uses three Lambda functions, each with a specific responsibility in the backup lifecycle:

### Backup Function
**Purpose**: Extract and store data from DynamoDB tables
- **Scans DynamoDB tables** to retrieve all records
- **Exports data to S3** in JSON format with timestamps
- **Generates checksums** for data integrity verification
- **Updates backup metadata** with operation details

### Validation Function  
**Purpose**: Verify backup integrity and completeness
- **Validates checksums** to ensure data wasn't corrupted
- **Verifies file completeness** by checking record counts
- **Updates backup status** in the metadata table
- **Triggers alerts** for any validation failures

### Recovery Function
**Purpose**: Restore data from backups when needed
- **Reads backup files** from S3 storage
- **Validates data integrity** before restoration
- **Populates DynamoDB tables** with restored data
- **Confirms successful restoration** with verification checks

### Content
1. [Backup Function](3.1.1-backupfunc/) 
2. [Validation Function](3.1.2-backupvalidator/) 
3. [Recovery Function](3.1.3-recoveryfunc/) 
