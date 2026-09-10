# networkwalks-B082-week4-Web-Application-Penetration-Testing-Project
├── exploit-poc/                # Automated exploit verification scripts
│   ├── sqli_auth_bypass.py     # Python automated payload script for Patient Portal
│   └── hash_extractor.py       # PDF metadata parsing utility
├── remediation-configs/        # Hardening configuration code patches
│   ├── httpd_security.conf     # Server configuration directives for LiteSpeed/Apache
│   └── secure_query.php        # Remediated PDO object handling function
└── README.md                   # Enterprise-ready project profile showcase
# 🔬 Advanced Infrastructure & Web Application Penetration Test (VAPT)
> **Target Entity:** Mediroza Enterprise Hospital Infrastructure (Staging Environment)  
> **Methodology Framework:** OWASP Web Security Testing Guide (WSTG v4.2)  
> **Assessment Scope:** Black-Box External Network + Grey-Box Web Application Auditing  

---

## 📊 Executive Threat Dashboard

During the authorized testing cycle against the Mediroza staging architecture, multiple structural gaps were discovered across the network boundary, server configuration layer, and database logic integration. The compilation of these flaws allowed an unauthenticated agent to pivot from network enumeration to full database exfiltration and patient record access.

| Assessment Component | Metrics / Targets Verified | Operational Risk Rating |
| :--- | :--- | :--- |
| **Network & Infrastructure** | Target IP: `199.188.201.16` / LiteSpeed Server | 🟡 **MEDIUM** (Information Disclosure) |
| **Application Layer Security** | Directory Traversal (`/old/`, `/patient/`) | 🔴 **CRITICAL** (Data Exposure) |
| **Authentication Integrity** | Patient Portal Login Validation Logic | 🔴 **CRITICAL** (Auth Bypass) |

---

## 🧭 Phase 1: Infrastructure Footprinting & Boundary Recon
Initial perimeter mapping focused on resolving public records and profiling edge handling nodes.

### 1. DNS Resolution Topology
Queries verified external boundaries pointing directly to target hosting ranges:
* **Host Address:** `199.188.201.16`
* **IPv6 Address Map:** `64:ff9b::c7bc:c910`

### 2. HTTP Banner Leakage
Manual connection validation utilizing raw `cURL` requests exposed exact backend platform versions, bypassing masking conventions:
```http
HTTP/2 200
content-type: text/html
server: LiteSpeed
x-turbo-charged-by: LiteSpeed
```
---

## 🔍 Phase 2: Technical Vulnerability Deconstruction & PoC

### Finding 1: Unauthenticated Directory Browsing & Data Exfiltration
* **Vulnerability Type:** CWE-548 (Exposure of Information Through Directory Listing) / CWE-522 (Insufficiently Protected Credentials)
* **Severity Score:** 🔴 **CRITICAL (CVSS v3.1: 9.8)**
* **Exploitation Vector:** Application configurations failed to restrict indexing on historical storage routes. Navigating to `/old/` exposed full administrative file systems, allowing direct downloads of raw production backups (`mediroza_db_backup_2019.sql`).
* **Compromised Assets:** Complete table structures and customer records were exposed in plaintext:
  * **`staff` Table:** Compromised names, roles (e.g., Medical Director, Chief Financial Officer), contact listings, unique `national_id` elements, and exact salaries mapped in ZAR currency.
  * **`shareholders` Table:** Exposed identity lists, corporate holding models (`Cedar Health Holdings (Pty) Ltd`), exact share counts, and structural classes.

### Finding 2: Structural SQL Injection (SQLi) Authentication Bypass
* **Vulnerability Type:** CWE-89 (Improper Neutralization of Special Elements used in an SQL Command)
* **Severity Score:** 🔴 **CRITICAL (CVSS v3.1: 10.0)**
* **Exploitation Vector:** Input forms at `/patient/login.php` passed parameters directly to database interpreters without validation checks. Inputting syntax characters (`admin'-- -`) forced the query logic to evaluate as always true, dropping password validations.
* **Automated PoC Script Verification (`exploit-poc/sqli_auth_bypass.py`):**
```python
import requests
target = "https://medirozahospital.com"
payload = {"username": "admin'-- -", "password": "password123"}

print(f"[*] Dispatching structural injection to: {target}")
session = requests.Session()
response = session.post(target, data=payload, allow_redirects=False)

if response.status_code == 302 and "portal.php" in response.headers.get("Location", ""):
    print("[+] Exploitation Successful: Session token generated via auth bypass.")
```

### Finding 3: Insecure Directory Architecture & Patient Record Access
* **Vulnerability Type:** CWE-219 (Storage of Files with Sensitive Data Under Web Root)
* **Severity Score:** 🟠 **HIGH (CVSS v3.1: 7.5)**
* **Exploitation Vector:** The system maps patient application panels under open indexes (`/patient/`). After leveraging the entry vulnerability, an operator gains access to `/patient/portal.php`, exposing password-encrypted clinical data sheets:
  * `Pathology Report - S. Dlamini` (Ref: LR-2024-1187)
  * `Pathology Report - P. Reddy` (Ref: LR-2024-1192)
  * `Pathology Report - E. Thompson` (Ref: LR-2024-1205)
    ---

## 🛡️ Phase 3: Defensive Remediation Engineering

### 1. Eliminating SQL Injection Vulnerabilities
Dynamic string compilation inside application code must be prohibited. Databases must consume input arguments exclusively through isolated parameters using **Prepared Statements**.

```php
// Remediated code pattern for integration into patient portal frameworks
\(db = new PDO('mysql:host=localhost;dbname=mediroza_hr;charset=utf8mb4',\)user, \(pass);\)db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

// Forcing strict execution boundary abstraction
\(query =\)db->prepare('SELECT id, password_hash FROM staff WHERE username = :user');
\(query->execute(['user' =>\)_POST['username']]);
\(account =\)query->fetch();
```

### 2. Disabling Global Directory Indexing
To secure sensitive application roots like `/old/` and `/patient/`, automatic resource index generations must be explicitly deactivated inside server configuration blocks.
 configuration for application entry boundaries
<Directory "/var/www/html/patient">
    Options -Indexes
</Directory>

<Directory "/var/www/html/old">
    Order deny,allow
    Deny from all
</Directory>
