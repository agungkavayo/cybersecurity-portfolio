# Host Enumeration & Secure Workspace Setup

## Objective
- Set up a structured and organized workspace  
- Identify and document host system information  
- Establish clear separation between multiple environments  
- Prevent operational errors during security assessments  


## Environment
| Host | Role | Description |
|------|------|------------|
| Ubuntu Lab | Primary | Used for workspace setup and main data collection |
| Kali Linux | Secondary | Used for validation and security-focused operations |


## Actions Taken
### 1. Workspace Initialization
A structured directory was created to organize all project-related files:
```bash
mkdir -p ~/module01/{notes,evidence,reports,tmp}
```
| Directory | Purpose                             |
| --------- | ----------------------------------- |
| notes     | Store observations and explanations |
| evidence  | Store raw command outputs           |
| reports   | Store summarized findings           |
| tmp       | Temporary files                     |

### 2. Host Enumeration
Basic system identity information was collected using the following command:
```bash
hostname && hostname -I && whoami && echo $SHELL && pwd
```
| Command     | Description                     |
| ----------- | ------------------------------- |
| hostname    | Displays system hostname        |
| hostname -I | Retrieves assigned IP address   |
| whoami      | Identifies current user         |
| echo $SHELL | Displays active shell           |
| pwd         | Shows current working directory |

## Key Output 
### Ubuntu Host
```
ubuntu-lab
10.10.10.x
student
/bin/bash
/home/student/module01
```

### Kali Linux Host
```
kali
10.10.10.y
kali
/bin/bash
/home/kali/module01
```

## What I Learned
Working across two hosts with different roles requires discipline from the start. Misidentifying which host you are on during an assessment can lead to incorrect results or scope violations. Establishing a baseline context early prevents this.
