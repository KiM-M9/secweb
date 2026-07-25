# SQL Injection Vulnerability

## Table of Contents

- [Overview](#overview)
- [How Does This Vulnerability Happen?](#how-does-this-vulnerability-happen)
  - [What Happened Here?](#what-happened-here)
- [Real-World Example: Login Bypass Lab (PortSwigger)](#real-world-example-login-bypass-lab-portswigger)
- [Vulnerable vs. Secure Code (PHP Example)](#vulnerable-vs-secure-code-php-example)
- [What Can an Attacker Achieve Through This Vulnerability?](#what-can-an-attacker-achieve-through-this-vulnerability)
- [Types of SQL Injection Vulnerabilities](#types-of-sql-injection-vulnerabilities)
- [How to Prevent SQL Injection](#how-to-prevent-sql-injection)
- [References](#references)

---

## Overview

**SQL Injection (SQLi)** is one of the oldest and most dangerous web application vulnerabilities. It occurs when user-supplied input is inserted directly into a SQL query without proper validation or sanitization, allowing an attacker to alter the logic of that query.

SQL Injection is consistently ranked among the [OWASP Top 10](https://owasp.org/www-project-top-ten/) most critical web application security risks, under the broader category of **Injection**. Despite being well understood, it remains common in real-world applications due to poor coding practices.

This vulnerability happens when a website lets a user enter data without filtering or validating it. The attacker exploits this weak point and injects malicious SQL code into the query.

---

## How Does This Vulnerability Happen?

Let's say there's a website with a login page, and the backend builds a query like this:

```sql
SELECT * FROM users WHERE username = 'input' AND password = 'input'
```

If the inputs aren't validated, an attacker can type something like:

```
' OR '1'='1
```

So the query turns into:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1'
```

### What Happened Here?

**AND has higher priority than OR**

In SQL (like most programming languages), the `AND` operator runs before the `OR` operator. So you have to imagine there are implicit parentheses around every `AND`:

```sql
WHERE username = '' 
   OR ('1'='1' AND password = '') 
   OR '1'='1'
```

- **First condition:** is the username empty? → `False`
- **Second condition:** `'1'='1'` and the password is empty → `False`, since the password can't actually be empty
- **Third condition:** `'1'='1'` → always `True`

Since the last part `OR '1'='1'` is a constant `True` that doesn't depend on any actual data in the database, the whole condition after `WHERE` becomes `True` for every row in the `users` table.

So the query effectively turns into:

```sql
SELECT * FROM users  -- returns every user, with no real filtering!
```

---

## Real-World Example: Login Bypass Lab (PortSwigger)

> Source: [PortSwigger Web Security Academy – Lab: SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

This lab contains a SQL injection vulnerability in the login function. The goal is to perform a SQL injection attack that logs in to the application as the `administrator` user — **without knowing the password**.

**Solution steps:**

1. Use **Burp Suite** to intercept and modify the login request.
2. Modify the `username` parameter, giving it the value:

```
administrator'--
```

**Why does this work?**

The backend query is likely something like:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'anything'
```

In SQL, `--` starts a comment, so everything after it (including the password check) is ignored:

```sql
SELECT * FROM users WHERE username = 'administrator'
```

The database returns the `administrator` row regardless of the password, and the application logs the attacker in as `administrator`.

---

## Vulnerable vs. Secure Code (PHP Example)

### Vulnerable Code

User input is concatenated directly into the SQL string — this is exactly what allows the attacks shown above.

```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = mysqli_query($connection, $query);

if (mysqli_num_rows($result) > 0) {
    echo "Login successful!";
} else {
    echo "Invalid credentials.";
}
?>
```

>  **Warning:** With input like `administrator'--`, this code lets an attacker log in without knowing the password.

###  Secure Code (Prepared Statements)

The input is treated strictly as **data**, never as executable SQL code.

```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

$stmt = $connection->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();

$result = $stmt->get_result();

if ($result->num_rows > 0) {
    echo "Login successful!";
} else {
    echo "Invalid credentials.";
}
?>
```
---

## What Can an Attacker Achieve Through This Vulnerability?

- Bypass authentication systems (log in without valid credentials)
- Steal data from the database (usernames, passwords, sensitive info)
- Modify or delete data
- In some cases, take full control of the server if the database permissions are too broad

---

## Types of SQL Injection Vulnerabilities

- **In-band SQLi**
  - Error-based SQLi
  - Union-based SQLi
- **Blind SQLi**
  - Boolean-based Blind
  - Time-based Blind
- **Out-of-band SQLi**
- **Second-Order SQLi**

We'll go over each type, understand what it is, and apply hands-on examples in upcoming sections.

---

## How to Prevent SQL Injection

| Technique | Description |
|---|---|
| **Prepared Statements / Parameterized Queries** | Separate SQL logic from user data so input can never change query structure (as shown in the PHP example above). |
| **Use an ORM** | Frameworks like Eloquent, Doctrine, or SQLAlchemy build queries safely and reduce raw SQL usage. |
| **Input Validation & Sanitization** | Validate input type, length, and format (e.g., allow only digits for numeric fields) as a secondary layer of defense — not a replacement for prepared statements. |
| **Least Privilege Principle** | Database accounts used by the application should have the minimum permissions needed (no `DROP`, `ALTER`, etc. unless required). |
| **Web Application Firewall (WAF)** | Can detect and block common injection patterns, adding an extra layer of protection. |
| **Error Handling** | Never expose raw database error messages to the user — they can leak schema details useful to an attacker. |
| **Regular Security Testing** | Use tools like Burp Suite, sqlmap, or manual code review to catch injection points before attackers do. |



---

## References

- [OWASP – SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PortSwigger Web Security Academy – SQL Injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger – Lab: SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)
