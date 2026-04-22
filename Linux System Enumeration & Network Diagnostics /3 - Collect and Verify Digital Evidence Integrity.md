# Digital Evidence Collection & Integrity Verification

## Objective

- Organize evidence into a structured workspace  
- Create an inventory of collected data  
- Archive artifacts into a single compressed file  
- Generate a SHA256 checksum for integrity verification  
- Establish a reproducible and verifiable evidence handling process  


## Environment
| System | Role | Description |
|--------|------|------------|
| Ubuntu Lab | Primary | Used for evidence collection, organization, and verification |


## Actions Taken

### 1. Workspace Organization

Evidence was categorized into structured directories to separate raw data, processed data, and archived results:
```bash
mkdir -p evidence/raw evidence/processed reports/archive
```
To maintain visibility and traceability, directory and file inventories were generated:
```
find ~/module01 -maxdepth 2 -type d | sort > workspace_tree.txt
find ~/module01/notes -maxdepth 2 -type f | sort > notes_inventory.txt
```

### 2. Archiving Process
All relevant files were compressed into a single archive for preservation and transfer:
```
tar -czf module01_workspace_initial.tar.gz ~/module01/notes ~/module01/evidence
```
This creates a snapshot of the workspace at a specific point in time.

### 3. Integrity Verification
A SHA256 checksum was generated to ensure the archive's integrity:
```
sha256sum module01_workspace_initial.tar.gz > archive_sha256.txt
```

### Sample Output
```
a3f92c1d4b...  module01_workspace_initial.tar.gz
```

## Key Finding
After hashing the archive, any future modification to the file — intentional or accidental — will produce a different hash. 
This makes the checksum a reliable and verifiable proof of integrity.

## What I Learned
Evidence that cannot be verified is evidence that cannot be trusted. In digital forensics, a SHA256 checksum is the minimum standard to establish that collected artifacts have not been altered after collection.
