# **Permission Management and Auditing in Linux (EN-Ver 🇺🇸🇬🇧)**

## **1. Introduction**
This document details the initial permission issues in the file system of the fictional company **Company_IT** and how they were corrected. Security principles and best practices in Linux system administration have been followed.

## **2. Initial Scenario**
The company's directory structure was disorganized, with excessive permissions granting unauthorized access to sensitive files, putting the integrity of the information and employee productivity at risk.

---

## **3. Directory Structure and Users**


"""
/company_IT/
├── departments/
│   ├── administration/  # Accessible only by the administration group
│   │   ├── contracts.xlxs
│   │   ├── hhrr_data.docx
│   │   └── payrolls.txt
│   ├── external/  # Read-only to prevent modifications
│   │   ├── customer_contracts.docx
│   │   └── supplier_contracts.txt
│   └── guest/  # Permission control for guests
│       ├── guest_env/
│       │   ├── index.html
│       │   ├── script.js
│       │   ├── styles.css
│       │   └── test.py
│       ├── guest_intern.txt
│       ├── jr_access.docx
│       └── sr_access.xlxs
├── projects/
│   ├── completed_projects.md
│   ├── projects_in_progress.py
│   └── version_control.git
└── resources/
    ├── backups.zip
    ├── repositories.git
    └── total_backup_resources.zip
"""
--

### **Users and Groups**

- **Users:** `administration`, `jr.`, `sr.` (Each with their own specific permissions).
- **Groups:** `guest`, `external`.
- **Special roles:** `IT_Admin`, `Admin_Manager` with global access to the company.

---

## **4. Initial Permissions Audit**

To review current permissions:

ls -la /company_IT/departments/

Example of problematic output:

drwxrwxrwx 5 root root 4096 Mar 14 10:09 administration
-rw-rw-r-- 1 user1 user1 2048 Mar 14 09:45 contracts.xlxs

Here we notice that *administration* has permissions of 777, which is a serious security flaw. We need to fix it.

---

## **5. Permissions Fix**

Changes are applied **without using octal notation**, only with verbose `chmod`:

### **Administration (Internal Access Only)**
```
chmod u=rwx,g=---,o=--- /company_IT/departments/administration/
chmod u=rw,g=---,o=--- /company_IT/departments/administration/*
```

Check:
```
ls -la /company_IT/departments/administration/
```
Expected Output:
```
drwx------ 5 administration administration 4096 Mar 14 10:09 administration
-rw------- 1 administration administration 2048 Mar 14 09:45 contracts.xlxs
```

### **External (Read Only)**
```
chmod u=rwx,g=r-x,o=r-x /company_IT/departments/external/
chmod u=rw,g=r--,o=r-- /company_IT/departments/external/*
```

### **Guest (Restrictions and Internship Environment)**
```
chmod u=rwx,g=rw-,o=--- /company_IT/departments/guest/
chmod u=rwx,g=rwx,o=--- /company_IT/departments/guest/guest_env/
chmod u=rw,g=r--,o=--- /company_IT/departments/guest/guest_intern.txt
```

### **Projects (Hierarchical Access)**
```
chmod u=rwx,g=r-x,o=--- /company_IT/projects/
chmod u=rw,g=r--,o=--- /company_IT/projects/completed_projects.md
chmod u=rw,g=rw,o=--- /company_IT/projects/projects_in_progress.py
```

---

## **6. Sticky Bit Implementation and Deletion Protection**

To prevent unauthorized users from deleting files:
```
chmod +t /company_IT/resources/
chmod +t /company_IT/projects/
```

To prevent writing to critical files:
```
chmod a-w /company_IT/resources/backups.zip
```

Verification:
```
ls -ld /company_IT/resources/
```
Expected output:
```
drwxrwxrwt 5 root root 4096 Mar 14 10:09 resources
```

--

## **7. Explanation of the Permission String**
The permission string consists of 10 characters:
- **drwxrwxrwx**

| Character | Description                      | Affects    |
|-----------|----------------------------------|------------|
| `d`       | Directory (or `-` for file)      | Type       |
| `rwx`     | User (Owner) permissions         | User       |
| `rwx`     | Group permissions                | Group      |
| `rwx`     | Others' permissions              | Others     |

- **User Permissions (`rwx`)**:
  - **r** (read): Allows viewing of file contents.
  - **w** (write): Allows modification of file or directory.
  - **x** (execute): Allows running the file or accessing the directory.

- **Group Permissions (`rwx`)**:
  - Same as user permissions, but applied to the group assigned to the file.

- **Others Permissions (`rwx`)**:
  - Same as user and group permissions, but applied to anyone else.


---

## **8. Hidden Files and Permissions**
Files that start with a dot (`.`) are considered hidden in Linux, and managing permissions for them works the same way as any other file.

To modify permissions on hidden files, use the same `chmod` command:

Example:
- **chmod u=rwx,g=rw-,o=--- /company_IT/departments/.hidden_file.txt**

- This command gives the owner full permissions, the group read and write access, and others no access.

- You can list hidden files by using the `ls -la` command, which will show all files, including hidden ones.


## **8. Summary**

Permissions have been corrected to ensure:
- Security for sensitive files (Administration, Resources).
- Controlled access for guests and external users.
- Hierarchy in `projects`.
- Protection against accidental deletion.

The system now follows the principle of **least privilege**, ensuring integrity and functionality.

--




