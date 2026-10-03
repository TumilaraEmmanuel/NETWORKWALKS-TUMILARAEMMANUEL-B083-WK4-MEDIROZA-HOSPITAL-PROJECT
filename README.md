# NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT

![Cybersecurity](https://img.shields.io/badge/Focus-Web%20Application%20Security-red)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-blue)
![Assessment](https://img.shields.io/badge/Assessment-Penetration%20Testing-orange)
![Status](https://img.shields.io/badge/Status-Completed-success)

### 👨‍💻 Pentester

**Opakunbi Oluwatumilara Emmanuel**

---

## 1. Executive Summary

This project involved an authorised penetration test of the **Mediroza General Hospital web application** as part of a cybersecurity training exercise.

I, **Opakunbi Oluwatumilara Emmanuel**, performed the assessment using a structured penetration-testing methodology covering reconnaissance, initial access, authentication testing, SQL injection testing, PDF security assessment, metadata analysis, exposed-directory investigation, and database-backup exposure.

The assessment demonstrated how several individual weaknesses could be chained together to create a significant security impact.

The overall attack path was:

```text
Reconnaissance
      ↓
robots.txt discovery
      ↓
Patient portal discovery
      ↓
Username enumeration
      ↓
SQL injection
      ↓
Authentication bypass
      ↓
Access to laboratory reports
      ↓
PDF password assessment
      ↓
PDF decryption
      ↓
Metadata analysis
      ↓
Discovery of /old/
      ↓
Exposed database backup
      ↓
Sensitive information exposure
```

The assessment identified vulnerabilities involving authentication, SQL injection, PDF protection, metadata leakage, directory listing, and publicly accessible database backups.

The project demonstrates the importance of secure authentication, input validation, proper access control, secure file handling, metadata sanitisation, and secure backup management.

> **Important:** This project was performed within an authorised educational penetration-testing environment. The techniques documented here must not be applied to systems without explicit written permission from the system owner.

---

# 2. Project Objectives

The primary objectives of the assessment were to:

* Perform reconnaissance against the authorised target.
* Identify hidden application directories.
* Identify the patient login portal.
* Test authentication behaviour.
* Identify username enumeration.
* Test the login functionality for SQL injection.
* Determine whether authentication could be bypassed.
* Assess access to laboratory reports.
* Assess the security of encrypted PDF reports.
* Analyse PDF metadata.
* Investigate clues discovered during metadata analysis.
* Identify exposed directories and backup files.
* Assess the security impact of an exposed database backup.
* Document the complete attack chain.
* Provide appropriate remediation recommendations.

---

# 3. Scope

### Target

```text
https://medirozahospital.com
```

### Primary Areas Assessed

```text
/patient/
/patient/login.php
/patient/reports/
/old/
```

### Key Files Encountered

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
mediroza_db_backup_2019.sql
```

The project solution identifies the target and the `/patient/`, `/staff/`, and `/old/` paths through the site's `robots.txt` file.

---

# 4. Tools and Technologies

The assessment involved:

| Tool / Technology             | Purpose                                             |
| ----------------------------- | --------------------------------------------------- |
| Kali Linux                    | Penetration-testing environment                     |
| `curl`                        | HTTP requests and reconnaissance                    |
| Browser                       | Web application interaction                         |
| Networkwalks Hash Calculator  | PDF hash extraction                                 |
| Networkwalks Password Cracker | Authorised PDF password assessment                  |
| JTR wordlist                  | Extended password assessment                        |
| `qpdf`                        | PDF decryption                                      |
| `exiftool`                    | PDF metadata analysis                               |
| `wget`                        | Downloading the authorised database backup          |
| ChatGPT                       | Converting authorised SQL data into readable tables |

---

# 5. Methodology

The assessment was completed through four main milestones:

```text
Milestone 1 — Initial Access
Milestone 2 — Crack the Encryption
Milestone 3 — Deep Reconnaissance
Milestone 4 — Penetration Testing Report
```

---

# 6. Milestone 1 — Initial Access

## Step 1 — Reconnaissance

The first stage was reconnaissance.

The site's `robots.txt` file was requested to identify directories that were not intended to be indexed by search engines.

### Kali Command

```bash
curl https://medirozahospital.com/robots.txt
```

### Result

```text
Disallow: /patient/
Disallow: /staff/
Disallow: /old/
```

This revealed three important directories:

```text
/patient/
/staff/
/old/
```

The `/patient/` directory became the focus of the initial-access phase, while `/old/` became relevant during the later deep-reconnaissance phase.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Curl%20Medirozahospital.com.png)

---

# 7. Step 2 — Locate the Login Page

The `/patient/` directory revealed the patient login page.

### Login URL

```text
https://medirozahospital.com/patient/login.php
```

The login page provided username and password fields.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Medirozah%20website%20(patient%20login).png)

The discovery of the login page provided the entry point for authentication testing.

---

# 8. Step 3 — Username Enumeration

The next stage tested whether the application disclosed whether a username existed.

## Test 1 — Invalid Username

```text
Username: bob
Password: test123
```

### Response

```text
Username not found
```

## Test 2 — Existing Username

```text
Username: admin
Password: test123
```

### Response

```text
Incorrect password
```

The two different responses revealed that `admin` was a valid account.

This represents a **username enumeration vulnerability** because the application provides different responses depending on whether the username exists.

### Security Impact

An attacker could use the difference in error messages to identify valid accounts before attempting further attacks.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Mediroza%20admin.png)

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Incorrect%20password.png)

The project solution identifies this as a **Medium-risk vulnerability**.

---

# 9. Step 4 — SQL Injection Testing

The login functionality was then tested for SQL injection.

A single quote was supplied in the username field.

### Test Input

```text
Username: admin'
Password: test123
```

### Response

```text
Warning: mysqli_query(): You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server
version for the right syntax to use near ''' at line 1
```

The database error demonstrated that user-controlled input was being inserted into a database query without appropriate protection.

### Security Impact

The error disclosed:

* Database technology information.
* SQL query processing behaviour.
* Evidence of unsafe input handling.
* A potential path toward authentication bypass.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/SQL%20Syntax%20error.png)

The project solution classifies the SQL injection vulnerability as **Critical**.

---

# 10. Step 5 — Authentication Bypass

The SQL injection vulnerability was then demonstrated to bypass the password condition.

### Authorised Test Payload

```text
admin' --
```

The SQL comment sequence causes the remainder of the query to be treated as a comment in the vulnerable application.

### Result

```text
Logged in.
```

The patient portal loaded and displayed three laboratory reports.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/username%20admin.png)

The project solution documents the resulting access to the patient portal and three reports.

---

# 11. Step 6 — Download the Laboratory Reports

Three PDF reports were available after accessing the portal:

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
```

These files were downloaded for the next stage of the assessment.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Result%20document%20download.png)

---

# 12. Milestone 2 — Crack the Encryption

The second milestone focused on assessing the password protection applied to the PDF reports.

---

# 13. Step 1 — Obtain PDF Hashes

PDF password protection was assessed by first obtaining a hash representation of each encrypted PDF.

The project used the Networkwalks Hash Calculator.

### Tool

```text
https://networkwalks.com/hash-calculator/
```

Each PDF was uploaded individually.

The generated hash began with:

```text
$pdf$
```

The hash was then used for password testing.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Hash%20Calculation%20for%20PDF%20result%201.png)

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/PDF%202%20Document%20HASH.png)

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/PDF%203%20HASH.png)


The project solution documents this process and explains that the hash acts as a mathematical representation of the encrypted PDF for password testing.

---

# 14. Step 2 — Password Assessment of Reports 1 and 2

A built-in common-password wordlist was used against the extracted hashes.

### Tool

```text
https://networkwalks.com/password-cracker/
```

### Documented Results

```text
patient_report_1.pdf → [REDACTED IN PUBLIC README]
patient_report_2.pdf → [REDACTED IN PUBLIC README]
```

The exercise demonstrated that weak passwords can be recovered through dictionary-based password testing.

### Security Lesson

Password-protected files are only as strong as the passwords protecting them.

Weak and commonly used passwords significantly reduce the security provided by encryption.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/PDF%201%20PASSWORD%20CRACKED.png)

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/PDF%202%20PASSWORD%20CRACKED.png)


The source documents successful password recovery for both reports.

---

# 15. Step 3 — Extended Password Assessment of Report 3

The initial common-password list did not recover the password for report 3.

### Initial Result

```text
Exhausted wordlist.
No match.
ACCESS DENIED.
```

A larger JTR wordlist supplied for the authorised exercise was then used.

### Result

The password was successfully recovered.

For public GitHub documentation, the actual password is intentionally omitted.

```text
patient_report_3.pdf → [REDACTED]
```

### Lesson

A failed small wordlist does not necessarily mean that a password is strong.

It may simply mean that the password is not contained within that particular wordlist.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Exhaused%20Wordlist.png)

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/PDF%203%20Password%20CRACKED.png)

The source explicitly documents the transition from the common-password list to the larger JTR list.

---

# 16. Step 4 — Create an Unlocked Copy of Report 3

After the password was recovered, `qpdf` was used to create an unlocked copy of the PDF.

### Kali Command

```bash
qpdf --password='<RECOVERED-LAB-PASSWORD>' --decrypt patient_report_3.pdf report3_open.pdf
```

The original encrypted file was preserved while a separate decrypted copy was created.

### Output

```text
report3_open.pdf
```

### Verify the file

```bash
ls -lh report3_open.pdf
```

The unlocked copy was required because the next stage involved detailed metadata analysis.

The source explicitly documents the `qpdf` command and explains why an unlocked copy was required.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f7e371086b954f5e4afbb58a684706a6ade0cb5c/qpdf%20open.png)

---

# 17. Milestone 3 — Deep Reconnaissance

The third milestone investigated information contained within the unlocked PDF and followed the clues discovered during the assessment.

---

# 18. Step 1 — PDF Metadata Analysis

`exiftool` was used to inspect metadata contained within the unlocked PDF.

### Kali Command

```bash
exiftool report3_open.pdf
```

The assessment examined metadata fields including:

* Author
* Creator
* Producer
* Creation information
* Modification information
* Comments
* Other embedded metadata

### Key Finding

The metadata contained information identifying an author and a comment referencing an old database backup.

The source records the following important clue:

```text
Author: j.malik
```

and:

```text
DB backup moved to /old before site migration, do not delete
```

### Security Significance

This was important because the metadata revealed information about internal file handling and pointed toward an old backup directory.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Mediroza%20hospital%20old.png)

The project solution documents both the author field and the embedded `/old` backup comment.

---

# 19. Step 2 — Correlate the Metadata With Reconnaissance

The `/old/` directory had already been discovered during the initial `robots.txt` reconnaissance.

This created an important correlation:

```text
robots.txt
    ↓
/old/
    ↓
PDF metadata
    ↓
Database backup clue
    ↓
/old/ directory
```

This demonstrates why information discovered during one stage of a penetration test should be retained and correlated with later findings.

The source explicitly identifies the relationship between the initial reconnaissance and the later metadata discovery.

---

# 20. Step 3 — Investigate the /old/ Directory

The discovered directory was accessed within the authorised testing environment.

### URL

```text
https://medirozahospital.com/old/
```

Directory listing was enabled.

The following backup was exposed:

```text
mediroza_db_backup_2019.sql
```

### Security Issue

The web server was allowing users to browse and access files stored inside an old backup directory.

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Mediroza%20hospital%20old%20db%20backup%20sql.png)

The source identifies directory listing as the configuration issue responsible for exposing the backup.

---

# 21. Step 4 — Download the Database Backup

The authorised project used:

```bash
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

This downloaded:

```text
mediroza_db_backup_2019.sql
```

### Verify the file

```bash
ls -lh mediroza_db_backup_2019.sql
```

### Identify the file

```bash
file mediroza_db_backup_2019.sql
```

The documented download command is provided directly in the project solution.

---

# 22. Step 5 — Review the Database Backup

The SQL backup contained database commands and data in raw text format.

The assessment focused on relevant authorised data rather than unnecessary extraction.

The important tables examined in the exercise included:

```text
staff
shareholders
```

### Staff Data

The relevant `INSERT INTO staff` records were extracted for controlled analysis.

The information was organised into:

```text
Name
Job Title
Department
Monthly Salary
```

### Shareholder Data

The relevant `INSERT INTO shareholders` records were organised into:

```text
Shareholder Name
Share Percentage
Share Class
```

### Screenshot

![image alt](https://github.com/TumilaraEmmanuel/NETWORKWALKS-TUMILARAEMMANUEL-B083-WK4-MEDIROZA-HOSPITAL-PROJECT/blob/f376f98d064ccd4b77e2cfb2ea1e7e6312bc0f90/Staff%20and%20Shareholder%20Details..png)

The source specifically instructs the analyst to convert the relevant SQL records into readable tables.

---

# 23. Step 6 — Correlate the Metadata and Staff Information

The final correlation linked the PDF metadata with the staff information contained within the exposed database backup.

The metadata identified:

```text
j.malik
```

The staff information identified the corresponding person as:

```text
Jameel Malik
IT Systems Administrator
```

This completed the information trail established by the assessment.

### Attack Chain

```text
PDF Metadata
     ↓
Author: j.malik
     ↓
/old/ clue
     ↓
/old/ directory
     ↓
Database backup
     ↓
Staff table
     ↓
Jameel Malik
     ↓
IT Systems Administrator
```

The source uses this correlation to demonstrate how seemingly minor information leakage can become significant when combined with another exposed resource.

---

# 24. Complete Attack Chain

The entire penetration-testing sequence can be represented as follows:

```text
01. robots.txt
        ↓
02. /patient/ discovered
        ↓
03. Patient login page
        ↓
04. Username enumeration
        ↓
05. Valid admin account identified
        ↓
06. SQL injection identified
        ↓
07. Authentication bypass
        ↓
08. Patient portal accessed
        ↓
09. Three PDF reports discovered
        ↓
10. PDF encryption assessed
        ↓
11. PDF hashes obtained
        ↓
12. Password assessment performed
        ↓
13. PDF passwords recovered
        ↓
14. qpdf used to create unlocked copy
        ↓
15. PDF metadata analysed
        ↓
16. /old/ backup clue discovered
        ↓
17. /old/ directory accessed
        ↓
18. Database backup exposed
        ↓
19. Database backup downloaded
        ↓
20. Staff/shareholder data identified
        ↓
21. Metadata and database information correlated
        ↓
22. Security findings documented
        ↓
23. Remediation recommendations developed
```

---

# 25. Findings Summary

| # | Vulnerability                                                    | Location                      | Risk     |
| - | ---------------------------------------------------------------- | ----------------------------- | -------- |
| 1 | Username enumeration                                             | `patient/login.php`           | Medium   |
| 2 | SQL injection / authentication bypass                            | `patient/login.php`           | Critical |
| 3 | Encrypted PDFs accessible after authentication bypass            | `patient/reports/`            | High     |
| 4 | Weak PDF passwords                                               | `patient_report_*.pdf`        | High     |
| 5 | Sensitive PDF metadata                                           | `patient_report_3.pdf`        | Medium   |
| 6 | Directory listing and exposed backup                             | `/old/`                       | Critical |
| 7 | Confidential staff/shareholder information in exposed SQL backup | `mediroza_db_backup_2019.sql` | Critical |

These risk classifications reproduce the findings summary in the supplied project solution.

---

# 26. Finding 1 — Username Enumeration

### Description

The login application returned different messages for an invalid username and an existing username.

### Evidence

```text
Invalid username:
Username not found

Existing username:
Incorrect password
```

### Impact

An attacker can determine whether a username exists and use the information to support subsequent attacks.

### Recommendation

Return the same generic authentication message for both invalid usernames and incorrect passwords.

Example:

```text
Invalid username or password.
```

The source recommends never revealing which authentication field failed.

---

# 27. Finding 2 — SQL Injection

### Description

The login form processed user input directly within a database query.

A single quote generated a SQL syntax error, demonstrating unsafe query construction.

### Impact

The vulnerability allowed authentication bypass and provided unauthorised access to the patient portal.

### Recommendation

Use:

* Parameterised queries.
* Prepared statements.
* Server-side input validation.
* Secure database access controls.

Never construct SQL queries directly from raw user input.

The project solution specifically recommends parameterised queries or prepared statements.

---

# 28. Finding 3 — Inadequate PDF Access Control

### Description

The laboratory reports became accessible after the authentication bypass.

### Impact

Unauthorised access to protected healthcare documents could expose sensitive information.

### Recommendation

* Store sensitive PDFs outside the public web root.
* Implement proper server-side access controls.
* Verify that every requested document belongs to the authenticated user.
* Use secure authorisation checks.
* Do not rely solely on unpredictable filenames or PDF passwords.

The supplied solution recommends storing PDFs outside the web root or behind appropriate access controls.

---

# 29. Finding 4 — Weak PDF Passwords

### Description

The PDF files were protected with passwords that could be recovered using password wordlists.

### Impact

Weak passwords reduce the effectiveness of file encryption.

### Recommendation

Use:

* Strong unique passwords.
* High-entropy credentials.
* Appropriate encryption.
* Secure key management.
* Separate access control from file-level encryption.

---

# 30. Finding 5 — Sensitive PDF Metadata

### Description

The unlocked PDF contained metadata that disclosed internal information.

A comment referenced the `/old` backup directory.

### Impact

Metadata can unintentionally expose:

* Internal usernames.
* Employee names.
* Internal processes.
* File history.
* Infrastructure clues.
* Backup locations.

### Recommendation

Remove unnecessary metadata before distributing sensitive documents.

The supplied project solution specifically recommends:

```bash
exiftool -all= filename.pdf
```

for metadata cleaning.

---

# 31. Finding 6 — Exposed Backup Directory

### Description

The `/old/` directory allowed directory listing and exposed an old database backup.

### Impact

An attacker could discover files that should not be publicly accessible.

### Recommendation

* Disable directory listing.
* Remove obsolete files.
* Remove old backups from the web root.
* Store backups outside publicly accessible directories.
* Restrict backup access using authentication and network controls.
* Regularly audit web directories for forgotten files.

The project solution specifically recommends disabling directory listing and removing or relocating old backup files.

---

# 32. Finding 7 — Exposed Database Backup

### Description

The publicly accessible SQL backup contained confidential staff and shareholder information.

### Impact

Exposure of the backup could lead to disclosure of sensitive organisational information.

Depending on the contents of a real production database, an exposed backup could potentially reveal:

* Employee information.
* Financial information.
* Account information.
* Database structure.
* Application configuration.
* Additional credentials or sensitive records.

### Recommendation

* Never store database backups in the public web directory.
* Encrypt backups at rest.
* Apply strict access controls.
* Maintain secure backup storage.
* Remove obsolete backups.
* Monitor access to backup repositories.
* Perform regular exposure checks.

---

# 33. Evidence and Screenshot Record

The following screenshots should be inserted into this README as evidence of each major stage.

| Screenshot | Evidence                                |
| ---------- | --------------------------------------- |
| 01         | `robots.txt` reconnaissance             |
| 02         | Patient login page                      |
| 03         | Username enumeration                    |
| 04         | SQL injection error                     |
| 05         | Authentication bypass                   |
| 06         | Three laboratory reports                |
| 07         | PDF hash extraction                     |
| 08         | Report 1 password assessment            |
| 09         | Report 2 password assessment            |
| 10         | Failed report 3 wordlist                |
| 11         | Successful extended wordlist assessment |
| 12         | `qpdf` decryption                       |
| 13         | PDF metadata                            |
| 14         | `/old/` directory listing               |
| 15         | Database backup                         |
| 16         | Final findings/remediation              |

---

# 34. Screenshot Evidence

## Reconnaissance

> **[INSERT SCREENSHOT]**

`robots.txt` showing discovered directories.

---

## Patient Login

> **[INSERT SCREENSHOT]**

Patient login page.

---

## Username Enumeration

> **[INSERT SCREENSHOT]**

Different responses for invalid and valid usernames.

---

## SQL Injection

> **[INSERT SCREENSHOT]**

SQL error generated by the vulnerable input.

---

## Authentication Bypass

> **[INSERT SCREENSHOT]**

Successful portal access.

---

## Laboratory Reports

> **[INSERT SCREENSHOT]**

Three reports available after authentication bypass.

---

## PDF Hash

> **[INSERT SCREENSHOT]**

Generated PDF hash.

---

## Password Assessment

> **[INSERT SCREENSHOT]**

Password assessment results.

---

## qpdf

> **[INSERT SCREENSHOT]**

`qpdf` command and unlocked PDF.

---

## Metadata

> **[INSERT SCREENSHOT]**

`exiftool` output.

---

## Exposed Backup

> **[INSERT SCREENSHOT]**

`/old/` directory listing.

---

## Database Backup

> **[INSERT SCREENSHOT]**

Authorised local inspection of the SQL backup.

---

# 35. Security Lessons Learned

This project demonstrated several important penetration-testing principles.

### 1. Reconnaissance matters

A seemingly simple `robots.txt` file provided information about hidden directories.

### 2. Small information leaks can become attack paths

Username enumeration provided useful information for further authentication testing.

### 3. Input validation is critical

The SQL error demonstrated that user input was reaching the database query unsafely.

### 4. Authentication must be properly implemented

A vulnerable login mechanism can expose everything behind the authentication boundary.

### 5. Encryption does not compensate for weak passwords

A strongly encrypted PDF can still be vulnerable when protected by a weak password.

### 6. Metadata should not be ignored

File metadata can contain internal information that assists reconnaissance.

### 7. Old files remain security risks

The `/old/` directory demonstrated how forgotten resources can become an entry point to sensitive information.

### 8. Backup files require strong protection

Database backups should never be exposed through a public web directory.

### 9. Vulnerabilities can be chained

The most important lesson from the project was not any individual vulnerability, but how multiple weaknesses could be connected.

```text
Information Disclosure
        +
SQL Injection
        +
Authentication Bypass
        +
Weak File Passwords
        +
Metadata Leakage
        +
Directory Listing
        +
Exposed Database Backup
        =
Significant Security Exposure
```

---

# 36. Remediation Summary

| Vulnerability           | Recommended Remediation                                            |
| ----------------------- | ------------------------------------------------------------------ |
| Username enumeration    | Use identical authentication error messages                        |
| SQL injection           | Use parameterised queries/prepared statements                      |
| Authentication bypass   | Correct server-side authentication and authorisation               |
| PDF access control      | Store sensitive files outside web root and enforce access controls |
| Weak PDF passwords      | Use strong, unique passwords and appropriate key management        |
| PDF metadata leakage    | Remove unnecessary metadata before distribution                    |
| Directory listing       | Disable directory listing                                          |
| Public backup           | Remove backups from public directories                             |
| Backup security         | Encrypt and restrict access to backups                             |
| Sensitive data exposure | Apply least privilege and data minimisation                        |

---

# 37. Recommended Secure Architecture

A safer implementation should follow this model:

```text
                    INTERNET
                       |
                       v
              Web Application
                       |
                Authentication
                       |
             Authorisation Check
                       |
              Patient's Own Data
                       |
             Secure File Storage
                       |
              Protected Database
                       |
              Encrypted Backups
                       |
             Restricted Backup
                 Infrastructure
```

Sensitive files and backups should never be directly exposed through publicly browsable web directories.

---

# 38. Project Skills Demonstrated

Through this project, I demonstrated practical experience with:

* Web application reconnaissance
* `robots.txt` analysis
* HTTP request analysis
* Authentication testing
* Username enumeration
* SQL injection identification
* Authentication-bypass assessment
* PDF security assessment
* Password-security assessment
* Wordlist-based password testing
* `qpdf`
* `exiftool`
* `curl`
* `wget`
* Linux command-line operations
* File and metadata analysis
* Database-backup exposure assessment
* Vulnerability documentation
* Risk classification
* Security remediation
* Attack-chain analysis
* Evidence documentation

---

# 39. Tools and Commands Reference

## Reconnaissance

```bash
curl https://medirozahospital.com/robots.txt
```

## PDF Decryption

```bash
qpdf --password='<RECOVERED-LAB-PASSWORD>' --decrypt patient_report_3.pdf report3_open.pdf
```

## PDF Metadata

```bash
exiftool report3_open.pdf
```

## Database Backup Download

```bash
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

## Metadata Cleaning

```bash
exiftool -all= filename.pdf
```

## These commands are the commands explicitly documented in the supplied project solution.

# 40. Suggested Repository Structure

```text
Mediroza-Pentest/
│
├── README.md
│
├── screenshots/
│   ├── 01-robots.txt.png
│   ├── 02-login-page.png
│   ├── 03-username-enumeration.png
│   ├── 04-sql-injection.png
│   ├── 05-auth-bypass.png
│   ├── 06-patient-reports.png
│   ├── 07-pdf-hash.png
│   ├── 08-report1-password.png
│   ├── 09-report2-password.png
│   ├── 10-report3-failed.png
│   ├── 11-report3-success.png
│   ├── 12-qpdf.png
│   ├── 13-metadata.png
│   ├── 14-old-directory.png
│   ├── 15-database-backup.png
│   └── 16-final-report.png
│
├── evidence/
│   └── redacted-evidence.txt
│
└── reports/
    └── penetration-test-report.pdf
```

---

# 41. Data Protection and Responsible Disclosure

This project involved a simulated/authorised healthcare environment and potentially sensitive information.

For a public GitHub portfolio:

* Do not publish real patient information.
* Do not publish real employee information.
* Do not publish recovered passwords.
* Do not publish authentication credentials.
* Do not publish database dumps.
* Do not publish private URLs or tokens.
* Redact sensitive screenshots.
* Replace sensitive information with placeholders.
* Keep original evidence in the authorised training environment.
* Only publish material approved for portfolio use.

The supplied project explicitly states that the target was authorised for security testing and that the techniques must not be applied to systems without explicit written permission.

---

# 42. Professional Conclusion

The Mediroza General Hospital penetration-testing exercise demonstrated how multiple security weaknesses can combine to produce a much larger security exposure than any single vulnerability would suggest.

The assessment began with simple reconnaissance through `robots.txt`, progressed to username enumeration and SQL injection, and resulted in authentication bypass and access to protected laboratory reports.

Further analysis of the reports revealed weak password protection and sensitive metadata. The metadata then provided a clue that connected the assessment back to an `/old/` directory discovered during the initial reconnaissance phase. Investigation of that directory revealed an exposed database backup containing confidential organisational information.

The project therefore demonstrated the complete penetration-testing process from:

```text
Reconnaissance
        ↓
Identification
        ↓
Validation
        ↓
Exploitation
        ↓
Post-access investigation
        ↓
Evidence collection
        ↓
Risk assessment
        ↓
Remediation
```

The most important professional lesson from this assessment is that effective security testing requires more than identifying isolated vulnerabilities. A penetration tester must understand how findings relate to one another, document the complete attack chain, assess the resulting impact, protect sensitive evidence, and provide practical remediation guidance.

---

# 43. Author

**Opakunbi Oluwatumilara**

Cybersecurity / Ethical Hacking Trainee

Areas demonstrated in this project:

```text
Web Application Security
Penetration Testing
Reconnaissance
Vulnerability Assessment
Linux
Kali Linux
SQL Injection
Authentication Security
PDF Security
Metadata Analysis
Information Disclosure
Security Reporting
```

---

## Disclaimer

This project was completed for authorised educational and cybersecurity training purposes.

All penetration-testing activities described in this repository were performed within the permitted project scope.

**Never perform penetration testing, password testing, authentication bypass, vulnerability exploitation, or data extraction against a system without explicit written authorisation from the owner.**
