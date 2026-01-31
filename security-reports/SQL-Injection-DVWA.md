# 🛡️ SQL Injection Vulnerability Assessment & Forensics Report

**Target Application:** Damn Vulnerable Web Application (DVWA)  
**Vulnerability Class:** OWASP Top 10 – A03: Injection  
**Assessment Type:** Controlled Lab Penetration Test  
**Author:** *David Rotshak*  
**Date:** *30th Jan 2026*  

---

## 1. Executive Summary

This report documents a successful identification, exploitation, and mitigation analysis of a **SQL Injection (SQLi)** vulnerability discovered during a controlled security assessment of the Damn Vulnerable Web Application (DVWA).

The vulnerability allowed an attacker to manipulate backend SQL queries, bypass application logic, and extract sensitive database records. This assessment demonstrates the complete vulnerability lifecycle: **detection, exploitation, impact analysis, and remediation**.

---

## 2. Scope & Environment

### 2.1 Target

- **Application:** Damn Vulnerable Web Application (DVWA)
- **Module Tested:** SQL Injection (User ID)
- **Security Level:** Low

### 2.2 Environment Setup

- **Host OS:** Linux
- **Containerization:** Docker
- **Web Server:** Apache
- **Backend Language:** PHP
- **Database Engine:** MariaDB (MySQL-compatible)
- **Access Method:** Web browser via `http://127.0.0.1`

---

## 3. Methodology

The assessment followed a **black-box testing approach**, simulating an external attacker with no prior knowledge of the application’s source code.

Testing phases included:

1. Input behavior analysis  
2. Error-based detection  
3. Boolean logic manipulation  
4. Data extraction  
5. Root cause analysis  
6. Mitigation review  

---

## 4. Vulnerability Discovery

### 4.1 Initial Injection Indicator

The following payload was submitted via the `id` parameter:

id=1'

**Observed Behavior:**

- The application returned a verbose SQL error message referencing **MariaDB**.
<img width="1200" height="218" alt="Ohmyfin" src="https://github.com/user-attachments/assets/88fb3e1c-2893-4298-ae9a-cfe232dd0c9a" />


**Conclusion:**

- User input is directly concatenated into an SQL query.
- Backend database identified as MariaDB/MySQL.
- Error-based SQL injection confirmed.

---

## 5. Exploitation Analysis

### 5.1 Boolean-Based SQL Injection

To confirm control over the SQL query logic, the following payload was submitted:

' OR '1'='1


**Result:**

- The application returned **all user records** from the database.
<img width="911" height="509" alt="2" src="https://github.com/user-attachments/assets/f503eb43-ac54-4b65-8abd-41f4f340d87f" />

### 5.2 Query Logic Reconstruction

The vulnerable SQL query was inferred as:

```sql
SELECT * FROM users WHERE id = '$id';
```

Injected payload altered the query logic to:

WHERE id = '' OR '1'='1'

Since the condition '1'='1' always evaluates to TRUE, the database returned all rows.

## 6. Impact Assessment
### 6.1 Security Impac

| Impact Area              | Severity |
| ------------------------ | -------- |
| Authentication bypass    | High     |
| Data disclosure          | High     |
| User credential exposure | High     |
| Database enumeration     | High     |

### 6.2 Risk Classification

- OWASP Category: A03 – Injection
- Estimated CVSS Score: High (8.0+)
- Exploitability: Easy
- User Interaction Required: None

## 7. Root Cause Analysis

The vulnerability exists due to the following insecure practices:

- Direct concatenation of user input into SQL queries
- Absence of prepared statements
- Lack of input validation
- Verbose database error messages
- Excessive database privileges

## 8. Mitigation & Remediation
### 8.1 Primary Mitigation: Prepared Statements

Vulnerable Code:
$query = "SELECT * FROM users WHERE id = '$id'";

Secure Implementation (PDO):
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
Prepared statements ensure user input is treated strictly as data and not executable SQL.

### 8.2 Input Validation (Defense-in-Depth)

Validate expected input types before processing:
if (!ctype_digit($id)) {
    die("Invalid input");
}

### 8.3 Least Privilege Database Access

- Avoid using administrative or root database accounts
- Restrict application database users to only required permissions:
- - SELECT
- - INSERT
- - UPDATE
This limits potential damage in the event of exploitation.

### 8.4 Error Handling & Information Disclosure

- Disable detailed SQL errors in production
- Log errors internally
- Display generic error messages to users
die("An unexpected error occurred.");

### 8.5 Additional Security Controls (Optional)

- Web Application Firewall (WAF)
- Centralized logging and monitoring
- Secure code reviews
- Automated security testing and dependency scanning

## 9. Lessons Learned

- SQL Injection cannot be prevented by input filtering alone
- Escaping characters is insufficient protection
- Secure database interaction design is critical
- Error messages significantly assist attackers
- Prepared statements effectively neutralize SQL injection attacks

## 10. Conclusion

This assessment demonstrates a full SQL Injection attack lifecycle within a controlled lab environment. The vulnerability was easily exploitable due to unsafe SQL query construction and inadequate security controls.

Implementing parameterized queries, strict input validation, least privilege principles, and secure error handling fully mitigates this class of vulnerability.

## 11. References

- OWASP Top 10 – A03: Injection
- OWASP SQL Injection Prevention Cheat Sheet
- MariaDB Documentation
- DVWA Official Documentation
