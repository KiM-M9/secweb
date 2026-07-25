
# In-Band SQL Injection (Classic SQLi)

In-band SQL Injection is the most common and straightforward type of SQLi. It occurs when the attacker uses the same communication channel to launch the attack and gather the results.

This type is divided into two main categories:

---
## 1. Error-Based SQL Injection
This technique relies on the application displaying detailed database error messages directly to the user instead of a generic error page. By intentionally sending inputs that trigger a SQL error, an attacker can extract critical information from the error output, such as:

* **Database Engine & Version** (e.g., MySQL, PostgreSQL, MSSQL)
* **Table and Column Names**
* **Database Schema Details**

> **Example:** Inputting a single quote (`'`) into an input field might return a detailed SQL syntax error instead of a standard *"An error occurred"* page. This confirms the vulnerability and provides actionable intel for further exploitation.
---
## 2. Union-Based SQL Injection
This technique leverages the `UNION` SQL operator, which allows combining the results of two or more `SELECT` statements into a single HTTP response. Attackers exploit this to append their own malicious query to pull sensitive data (such as user credentials) from other tables.

* **Prerequisite:** The injected query must match the original query in both the **number of columns** and their corresponding **data types**.
* **Methodology:** Attackers typically determine the correct number of columns incrementally (e.g., using `ORDER BY` or `UNION SELECT NULL...`) before executing the payload.
