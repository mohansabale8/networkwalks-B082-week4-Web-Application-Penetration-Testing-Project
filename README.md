
  # 🔐 Web Application Penetration Testing — Week 4

<p align="center">
  <img src="https://img.shields.io/badge/Project-Penetration%20Testing-0A66C2?style=for-the-badge">
  <img src="https://img.shields.io/badge/Platform-Kali%20Linux-557C94?style=for-the-badge">
  <img src="https://img.shields.io/badge/Focus-Web%20Application%20Security-1F8ACB?style=for-the-badge">
  <img src="https://img.shields.io/badge/Status-Completed-success?style=for-the-badge">
</p>

<p align="center">
  <b>Cybersecurity Internship — Week 4 Project</b><br>
  Web Application Security Assessment
</p>

---

## 📌 Project Overview

This repository documents my Week 4 cybersecurity internship project focused on **web application penetration testing**.

The assessment followed a structured security-testing methodology covering:

- 🔎 Reconnaissance
- 🌐 Web enumeration
- 🧩 Technology identification
- 🔐 Authentication testing
- 💉 SQL injection assessment
- 🔑 Password security analysis
- 📸 Evidence collection
- 🛡️ Risk analysis
- 🔧 Remediation recommendations

> ⚠️ **Testing was performed only within the authorized internship assessment scope, against a lab/training target.**

---

# 🎯 Objectives

The main objectives of this assessment were to:

- Understand the target application's external attack surface
- Gather publicly available information
- Identify technologies used by the application
- Enumerate accessible application functionality
- Assess authentication mechanisms
- Identify input-validation weaknesses
- Validate discovered vulnerabilities in a controlled environment
- Analyze password security
- Document findings with screenshots
- Recommend appropriate security controls

---

# 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| 🐧 Kali Linux | Security testing environment |
| 🌐 `curl` | HTTP response and header inspection |
| 🔍 `Whois` | Domain information gathering |
| 🌎 `dig` / `nslookup` | DNS reconnaissance |
| 🕵️ `WhatWeb` | Web technology fingerprinting |
| 🧱 `WAFW00F` | WAF detection |
| 🔐 Network Walks Password Cracker | Dictionary-attack password recovery for encrypted PDFs |
| #️⃣ Hash Calculator | PDF hash ($pdf$) extraction for cracking |
| 🗄️ MySQL / `mysqldump` | Database enumeration and data-exposure evidence |
| 🌐 Web Browser | Application testing and verification |

---

# 🧭 Methodology

The assessment followed the workflow below:

```text
┌─────────────────────┐
│   Reconnaissance    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Attack Surface Map  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│    Enumeration      │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Vulnerability Test  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Controlled Validate │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Impact Analysis   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│    Remediation      │
└─────────────────────┘
```

---

# 🔎 01 — Reconnaissance

Reconnaissance was used to establish an initial understanding of the target environment before application-level testing.

## 🌐 1.1 Whois Enumeration

```bash
whois medirozahospital.com
```

Whois enumeration was used to identify publicly available domain and registration information.

## 🌎 1.2 DNS Reconnaissance

```bash
dig medirozahospital.com
nslookup medirozahospital.com
```

DNS reconnaissance was used to identify DNS records, name servers, resolved addresses, and other publicly exposed DNS information.

## 🕵️ 1.3 Technology Fingerprinting

```bash
whatweb https://medirozahospital.com
```

WhatWeb was used to identify the web server, frameworks, CMS indicators, and HTTP headers exposed by the application.

## 🧱 1.4 WAF Detection

```bash
wafw00f https://medirozahospital.com
```

WAFW00F was used to identify whether a recognizable Web Application Firewall was protecting the target.

---

# 🌐 02 — HTTP Inspection

```bash
curl -I https://medirozahospital.com
```

`curl` was used to inspect the HTTP response headers and identify information exposed by the web server (status, server banner, content type, security headers).

---

# 🗺️ 03 — Web Application Enumeration

After the initial reconnaissance phase, application functionality was reviewed to understand the available attack surface, including login interfaces, patient functionality, staff functionality, public resources, and download functionality.

---

# 🔐 04 — Authentication Testing

## 4.1 Patient Login Assessment

The patient login functionality was manually assessed for improper handling of user-controlled input.

During testing, an unexpected single quote was entered into the username field. The application returned a database-related error:

```text
Warning: mysqli_query(): You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version
for the right syntax to use near ''' at line 1
```

### 🚨 Observation

The application was exposing a raw MySQL error to the client, indicating that the supplied input was reaching a database query without safe handling.

---

# 💉 05 — SQL Injection Assessment

## 5.1 Vulnerability Identification

The database error observed during authentication testing indicated possible SQL injection. Further controlled testing demonstrated that specially crafted input could affect the authentication query.

## 5.2 Authentication Bypass

The authentication mechanism was successfully bypassed during the authorized assessment, demonstrating that authentication could be affected without providing the legitimate account password.

```text
Login Page → Input Testing → Database Syntax Error → SQL Injection Suspected → Controlled Validation → Authentication Bypass
```

## 5.3 Database Enumeration Evidence

While testing the patient login, entering a single quote in the username field produced a MySQL syntax error, indicating that the input was being processed directly by a database query. Further testing with `admin'--` successfully bypassed the authentication mechanism. After obtaining authenticated access, the application's patient-report functionality became accessible, allowing us to view confidential patient reports. This demonstrated how the SQL injection vulnerability could lead to unauthorized access to sensitive patient information.

Following the SQL injection finding, database contents were enumerated to assess the real-world impact of the vulnerability. Extracted table structures included a `shareholders` table and a `staff` table, both of which contained sensitive business and personal data.

# 👤 06 — Staff Authentication

The staff authentication interface was reviewed as part of the authentication-security assessment, covering login form behavior, input handling, authentication response, and error handling.

---


# 🔑 07 — Password Security Analysis

Protected PDF lab reports retrieved from the patient portal were assessed for password strength using a dictionary-attack password cracker.

## 7.1 Password Analysis Workflow

```text
Protected Material → Identify Hash/Format → Hash Calculator → Password Cracker → Controlled Validation
```

## 7.2 Protected Material

The patient portal exposed a list of password-protected lab report PDFs available for download:

![List of password-protected lab reports](password/accessed-files.png)

## 7.3 Password Cracker — Dictionary Attack Results

Each PDF's `$pdf$` hash was extracted and run through a dictionary attack. All three tested files used weak, easily guessable passwords:

**Report 1 — cracked password: `123456`**

![First password cracked - 123456](password/first-password.png)

**Report 2 — cracked password: `password`**

![Second password cracked - password](password/second-password.png)

**Report 3 — cracked password: `!@#$%^&`**

![Third password cracked - special characters](/password/third-password.png)

## 7.4 Verification — Decrypted Reports

The recovered passwords successfully unlocked the corresponding PDFs, confirming the weakness. These reports contained confidential patient health information:

![Cracked pathology report - Sipho Dlamini](/password/cracked-pdf-1.png)

![Cracked pathology report - Emily Thompson (1)](/password/cracked-pdf-2.png)

![Cracked pathology report - Emily Thompson (2)](/password/cracked-pdf-3.png)

> 🔒 **These reports contain real-format patient health information (names, DOB, results). Redact patient-identifying fields before publishing this repository publicly.**

---

# 🗄️ Backup Database Discovery

During reconnaissance, `curl` was used to inspect the website and its accessible directories. Further enumeration revealed an old `/old/` directory containing an exposed database backup. Examining the backup revealed the application's database structure and helped identify tables and fields related to the application.

> 🔒 Confidential patient information has been redacted from all publicly shared screenshots.

**Shareholders table dump:**

![Shareholders table dump](sqli/shareholders-table.png)

**Staff table dump** — note this table exposed staff national ID numbers and salaries in plaintext, which significantly raises the severity of this finding:

![Staff table dump](sqli/staff-table.png)

> 🔒 **These tables contain PII (national ID numbers) and confidential HR data (salaries, share ownership). Redact these fields before publishing this repository publicly, or replace the screenshots with cropped/blurred versions.**

---


# 📊 08 — Findings Summary

| ID     | Finding                             | Severity    | Status               |
| ------ | ------------------------------------ | ----------- | --------------------- |
| WEB-01 | SQL Injection in Authentication      | 🔴 Critical | Confirmed              |
| WEB-02 | Authentication Bypass                | 🔴 Critical | Confirmed              |
| WEB-03 | Database Error Disclosure            | 🟠 High     | Confirmed              |
| WEB-04 | Sensitive Data Exposure via SQLi     | 🔴 Critical | Confirmed              |
| WEB-05 | Weak PDF Password Protection         | 🟠 High     | Confirmed              |
| WEB-06 | Reconnaissance Information Exposure  | 🟡 Medium   | Assessment dependent   |

---

# 💥 09 — Impact Assessment

The authentication vulnerability demonstrates a significant weakness in the application's security boundary. Combined with the database enumeration and weak PDF password protection, the potential consequences include:

- Unauthorized authentication as any patient or staff account
- Full read access to the `shareholders` and `staff` database tables, including national ID numbers and salary data
- Unauthorized access to confidential patient pathology reports
- Exposure of business-sensitive shareholding information
- Reputational and regulatory (POPIA/data-protection) risk to the organization

The actual impact should be limited to what was demonstrated within the authorized assessment environment.

---

# 🛡️ 10 — Remediation

## 💉 SQL Injection

The application should use **prepared statements / parameterized queries** instead of dynamically constructing SQL queries from user input.

```text
User Input → Input Validation → Parameterized Query → Database
```

## 🔐 Authentication Security

Recommended controls include:

- Strong server-side authentication
- Secure password hashing
- Prepared SQL statements
- Login rate limiting
- Account lockout protections where appropriate
- Secure session management
- Authentication logging
- MFA for privileged accounts

## 🚫 Error Handling

Database errors should not be displayed directly to users. Instead of exposing internal database information, the application should return a generic message such as:

```text
Unable to process your request.
```

Detailed technical errors should remain in server-side logs.

## 🔑 Password Security Recommendations

- Enforce long, unique, system-generated passwords for protected PDF reports rather than weak defaults
- Apply strong password policies for all accounts
- Use secure, salted password hashing
- Protect against brute-force / dictionary attacks (rate limiting, lockouts)
- Enable MFA for sensitive/staff accounts
- Conduct regular credential-security reviews

## 🗄️ Database Access Control

- Restrict database accounts to least-privilege access
- Encrypt sensitive fields at rest (national ID numbers, salaries)
- Apply strict input validation and output encoding across all query paths
- Monitor and alert on abnormal or bulk data-extraction queries

---

# 📸 11 — Evidence Gallery

```text
screenshots/
├── sqli/
│   ├── shareholders-table.png
│   └── staff-table.png
│
└── password/
    ├── accessed-files.png
    ├── first-password.png
    ├── second-password.png
    ├── third-password.png
    ├── cracked-pdf-1.png
    ├── cracked-pdf-2.png
    └── cracked-pdf-3.png
```

---

# 📚 12 — Key Learnings

### 🔎 Reconnaissance
Understanding the target before testing helps build an accurate attack-surface map.

### 🌐 Enumeration
Small pieces of publicly available information can reveal useful application functionality.

### 💉 SQL Injection
Improperly handled user input can allow an attacker to influence database queries — and, as shown here, to fully enumerate sensitive tables.

### 🔐 Authentication Security
A weakness in the authentication layer can have a much greater impact than a vulnerability affecting a single page.

### 🔑 Password Security
Weak password protection can significantly reduce the security of otherwise protected information, as demonstrated by cracking all three sampled PDF passwords in seconds using a small dictionary.

### 📸 Evidence Collection
A professional security assessment should include reproducible evidence for significant findings.

---

# 📝 13 — Conclusion

The Week 4 assessment provided hands-on experience with the penetration-testing lifecycle, from reconnaissance and enumeration through vulnerability identification, controlled validation, impact assessment, and remediation.

The most significant findings were an authentication-related SQL injection that allowed authentication bypass and full enumeration of the `shareholders`/`staff` tables, plus weak password protection on downloadable patient PDF reports.

The assessment highlighted the importance of:

- Secure input handling
- Parameterized database queries
- Proper authentication controls
- Secure error handling
- Strong password protection
- Evidence-based security reporting

---

# ⚠️ Responsible Disclosure & Testing

This project was conducted as part of an authorized cybersecurity internship assessment.

All testing should only be performed against systems for which explicit authorization has been granted.

Sensitive information such as:

- Passwords
- Session tokens
- Personal information
- Patient information
- Private files
- Authentication hashes
- National ID numbers and salary data

should be removed or redacted before public publication of this repository.

---

# 👩💻 Author

**Mohan Sabale**

Cybersecurity Intern

### Skills Practiced

`Reconnaissance` • `Web Security` • `SQL Injection` • `Authentication Testing` • `Password Analysis` • `Linux` • `Security Reporting`

---

<p align="center">

### 🔐 Learn • Test • Document • Secure

</p>

