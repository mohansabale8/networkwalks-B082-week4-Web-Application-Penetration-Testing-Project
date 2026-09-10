# 🔐 Advanced Web Application Penetration Testing Portfolio
### Target: Mediroza Hospital Web Application (Simulated Environment)

---

## 📌 Executive Summary
This repository documents a comprehensive, methodology-driven web application penetration testing assessment executed against the **Mediroza Hospital** application. The primary objective was to evaluate the overall security posture, identify technical vulnerabilities, assess potential business impacts, and provide actionable remediation guidelines. 

Testing was strictly confined to an authorized training laboratory environment mimicking real-world production configurations.

### 📊 Security Posture Overview

| Metric | Assessment Summary |
| :--- | :--- |
| **Assessment Type** | Web Application Penetration Testing (Black Box/Grey Box) |
| **Target Infrastructure** | `https://medirozahospital.com` (Staging Environment) |
| **Critical Findings** | 3 |
| **High Findings** | 2 |
| **Remediation Priority** | **CRITICAL** (Requires immediate input validation overhaul) |

---

## 🧭 Assessment Methodology
The assessment follows an optimized security testing framework heavily aligned with the **OWASP Web Security Testing Guide (WSTG)** and standard PTES phases.

---

## 🛠️ Specialized Toolchain

| Tool | Focus Area | Technical Application |
| :--- | :--- | :--- |
| **Kali Linux** | Operating Platform | Centralized, secure penetration testing environment. |
| **cURL** | Traffic Inspection | Manual raw HTTP response header analysis. |
| **WhatWeb / WAFW00F** | Fingerprinting | Infrastructure mapping and Web Application Firewall detection. |
| **Hashcat / John** | Cryptanalysis | High-throughput dictionary attack mapping against PDF hashes. |
| **MySQL Toolkit** | Database Auditing | Extracting schema layouts and validating data exfiltration vectors. |

---

## 🔍 Detailed Phase Execution & Technical Findings

### Phase 1: Passive & Active Reconnaissance
Initial boundary mapping focused on gathering DNS footprinting information and service identification.
* **DNS Reconnaissance:** Utilizing `dig` and `nslookup` to enumerate name servers and trace domain validation vectors.
* **Technology Fingerprinting:** 
  ```bash
  whatweb https://medirozahospital.com
  wafw00f https://medirozahospital.com
  ```

### Phase 2: Vulnerability Analysis & Exploitation (Proof of Concept)

#### 🚨 Finding 1: SQL Injection (SQLi) in Patient Portal Authentication
* **Vulnerability Identifier:** CWE-89 (Improper Neutralization of Special Elements used in an SQL Command)
* **Severity:** 🔴 **CRITICAL**
* **Technical Description:** Entering an unescaped single quote (`'`) into the patient login username field triggered a raw MySQL error, confirming that user inputs are directly concatenated into the SQL statement block.
* **Raw Error Discovered:**
  ```sql
  Warning: mysqli_query(): You have an error in your SQL syntax; check the manual... near ''' at line 1
  ```
* **Exploitation / Auth Bypass Vector:**
  Passing a structural tautology completely bypassed authentication controls without valid credentials:
  ```text
  Username: admin'-- -
  Password: [Arbitrary Value]
  ```

#### 🚨 Finding 2: Unauthenticated Directory Traversal & Sensitive Data Leakage
* **Vulnerability Identifier:** CWE-522 (Insufficiently Protected Credentials) / CWE-219 (Storage of Files with Sensitive Data Under Web Root)
* **Severity:** 🔴 **CRITICAL**
* **Technical Description:** A directory enumeration discovery exposed an archived `/old/` legacy folder containing plaintext `.sql` file dumps.
* **Impact Summary:** Complete compromise of corporate schema infrastructure including the `shareholders` and `staff` rosters (containing plaintext salaries and national identifier keys).

#### 🚨 Finding 3: Insecure Document Cryptography (Weak PDF Passwords)
* **Vulnerability Identifier:** CWE-326 (Inadequate Encryption Strength)
* **Severity:** 🟠 **HIGH**
* **Technical Description:** Patient pathology records extracted via the authenticated layer were encrypted with trivial, dictionary-vulnerable passwords.
* **Cryptanalysis Vector:** PDF hashes were extracted utilizing a Python-based Hash Calculator utility and run against common wordlists.
* **Cracked Outputs:**
  * Report 1: `123456`
  * Report 2: `password`
  * Report 3: `!@#$%^&`

---

## 🛡️ Strategic Vulnerability Remediation Matrix

### 1. SQL Injection Eradication
**Defensive Strategy:** Transition the backend completely away from dynamic query concatenation. Enforce **Parameterized Queries / Prepared Statements** globally.
* **Remediation Sample (PHP Data Objects):**
  ```php
  $stmt = $pdo->prepare('SELECT id, password_hash FROM staff WHERE username = :username');
  $stmt->execute(['username' => $userInput]);
  $user = $stmt->fetch();
  ```

### 2. Information Disclosure Prevention
**Defensive Strategy:** Modify error handling architectures. Disable raw stack traces or SQL error feedback inside client-facing HTTP responses. Enforce generic UI error exceptions while logging technical exceptions securely on the server-side.
```text
Generic Error Output: "An unexpected error occurred. Please contact system support."
```

### 3. Structural Cryptographic Upgrades
**Defensive Strategy:** Enforce a strict password complexity configuration blueprint for programmatically compiled documents (minimum 16-character randomized alpha-numeric tokens). Encrypt highly sensitive PII values (Salaries, National ID keys) directly inside the database tier at rest utilizing AES-256.

---

## 📝 Continuous Professional Conclusions
The vulnerabilities discovered on the Mediroza network represent highly exploitable vectors that, if leveraged by real-world adversaries, could induce massive compliance violations (e.g., POPIA, GDPR, HIPAA) alongside immense reputational damage. Resolving these core findings requires a fundamental shift toward defensive coding principles, strict input validation routines, and the continuous auditing of public asset storage.
