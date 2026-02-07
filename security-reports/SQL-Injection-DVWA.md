# 🛡️ SQL Injection Vulnerability Assessment & Forensics Report

**Target Application:** Damn Vulnerable Web Application (DVWA)
**Vulnerability Class:** OWASP Top 10 – A03: Injection
**Assessment Type:** Controlled Lab Penetration Test (Educational)
**Primary Tooling:** Burp Suite (Proxy, Repeater)
**Author:** David Rotshak
**Date:** 30 January 2026

---

## 1. Executive Summary

This report documents the identification, confirmation, and exploitation of an **error‑based and boolean‑based SQL Injection (SQLi)** vulnerability within the Damn Vulnerable Web Application (DVWA).

The assessment demonstrates how **Burp Suite** was used as an interception and analysis proxy to capture HTTP requests and responses, validate database interaction, and confirm exploitability through controlled payloads.

The vulnerability allowed manipulation of backend SQL queries, resulting in **unauthorized data disclosure** and complete compromise of the application’s database query logic.

---

## 2. Scope & Environment

### 2.1 Target Scope

* **Application:** Damn Vulnerable Web Application (DVWA)
* **Module Tested:** SQL Injection (User ID)
* **Security Level:** Low
* **Access Type:** Authenticated user session

### 2.2 Test Environment

| Component        | Details                              |
| ---------------- | ------------------------------------ |
| Host OS          | Linux                                |
| Deployment       | Docker                               |
| Web Server       | Apache 2.4.25                        |
| Backend Language | PHP                                  |
| Database         | MariaDB (MySQL‑compatible)           |
| Client           | Chromium Browser                     |
| Proxy Tool       | Burp Suite Community                 |
| Target URL       | [http://127.0.0.1](http://127.0.0.1) |

---

## 3. Methodology

A **black‑box testing methodology** was followed, simulating an external attacker with no access to application source code.

Burp Suite was configured as an **intercepting proxy** between the browser and the DVWA instance, allowing full visibility of:

* Outgoing HTTP requests
* Incoming server responses
* Session cookies and parameters
* Database error messages

### Testing Phases

1. Baseline request capture (normal input)
2. Error‑based SQL injection probing
3. Boolean‑based logic manipulation
4. Response comparison and validation
5. Root cause analysis
6. Mitigation review

---

## 4. Vulnerability Discovery

### 4.1 Request Interception (Burp Suite)

Using **Burp Proxy → HTTP History**, a legitimate request was captured:

```
GET /vulnerabilities/sqli/?id=1&Submit=Submit HTTP/1.1
```

This request was forwarded to **Burp Repeater** to allow controlled manipulation of the `id` parameter.

---

### 4.2 Error‑Based SQL Injection Probe

#### Payload Submitted

```
id=1'
```

#### Observed Response (Captured in Burp)

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version...
```

#### Analysis

* The database engine (**MariaDB**) generated a syntax error
* The error message was returned in the HTTP response
* This confirms that user input was directly concatenated into a SQL query

**Conclusion:** Error‑based SQL injection confirmed.

---

## 5. Exploitation Analysis

### 5.1 Boolean‑Based SQL Injection

To verify full control over query logic, a boolean‑based payload was used.

#### Payload

```
' OR '1'='1
```

#### Result

* Application returned **all rows** from the `users` table
* Normal query restrictions were bypassed

This confirms that injected SQL logic was executed by the database.

---

### 5.2 Inferred Vulnerable Query

Based on response behavior, the backend SQL query was reconstructed as:

```sql
SELECT * FROM users WHERE id = '$id';
```

Injected payload modified the query to:

```sql
SELECT * FROM users WHERE id = '' OR '1'='1';
```

Since the condition `'1'='1'` always evaluates to TRUE, the database returned all records.

---

## 6. Impact Assessment

### 6.1 Security Impact

| Impact Area               | Severity |
| ------------------------- | -------- |
| Authentication bypass     | High     |
| Sensitive data disclosure | High     |
| User credential exposure  | High     |
| Database enumeration      | High     |
| Application integrity     | High     |

### 6.2 Risk Classification

* **OWASP Category:** A03 – Injection
* **Estimated CVSS Score:** 8.0 (High)
* **Exploit Complexity:** Low
* **User Interaction:** None
* **Attack Vector:** Remote (HTTP)

---

## 7. Root Cause Analysis

The vulnerability exists due to multiple insecure development practices:

* Direct concatenation of user input into SQL queries
* Absence of prepared statements
* No server‑side input validation
* Verbose database error disclosure
* Excessive database privileges

---

## 8. Mitigation & Remediation

### 8.1 Primary Fix: Parameterized Queries

#### Vulnerable Code

```php
$query = "SELECT * FROM users WHERE id = '$id'";
```

#### Secure Implementation (PDO)

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
```

Prepared statements ensure user input is treated strictly as data.

---

### 8.2 Input Validation (Defense‑in‑Depth)

```php
if (!ctype_digit($id)) {
    die("Invalid input");
}
```

---

### 8.3 Least‑Privilege Database Access

* Avoid root or admin database users
* Restrict application accounts to required permissions only:

  * SELECT
  * INSERT
  * UPDATE

---

### 8.4 Error Handling & Information Disclosure

* Disable SQL error display in production
* Log errors internally
* Display generic error messages to users

```php
die("An unexpected error occurred");
```

---

### 8.5 Additional Security Controls

* Web Application Firewall (WAF)
* Centralized logging and monitoring
* Secure code reviews
* Automated security testing (DAST/SAST)

---

## 9. Lessons Learned

* SQL injection is a design flaw, not an input‑filtering issue
* Error messages significantly reduce attack complexity
* Burp Suite is invaluable for request/response correlation
* Prepared statements fully neutralize SQL injection attacks
* Authentication gates must be tested alongside input handling

---

## 10. Conclusion

This assessment demonstrates a complete SQL injection attack lifecycle within a controlled laboratory environment using **Burp Suite** for interception and analysis.

The vulnerability was trivially exploitable due to unsafe SQL query construction and insufficient defensive controls. Implementing parameterized queries, strict input validation, least‑privilege access, and secure error handling fully mitigates this vulnerability class.

---

## 11. References

* OWASP Top 10 – A03: Injection
* OWASP SQL Injection Prevention Cheat Sheet
* MariaDB Documentation
* DVWA Official Documentation

