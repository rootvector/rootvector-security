# SQL Injection — Blind SQL Injection with Conditional Responses

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-exploiting-blind-sql-injection-by-triggering-conditional-responses/sql-injection/blind/lab-conditional-responses)

## Objective

This lab demonstrates a **blind SQL injection** vulnerability where the application does not directly display the result of the injected query.

Instead, the response changes depending on whether an injected SQL condition is **true or false**. The lab can therefore be solved by observing the application's conditional response and progressively determining the administrator password.

The objective is to identify the `administrator` account, determine the password length, recover the password character by character, and use the discovered credentials to log in.

---

## Initial Reconnaissance

During testing, I observed that the application's HTTP response contains a `"Welcome back"` message when the injected SQL condition evaluates to true.

A false condition causes the response to omit the `"Welcome back"` message.

This difference in the HTTP response provides the Boolean signal needed to exploit the blind SQL injection.

---

## Interesting Parameter

The username input was used to inject SQL conditions.

The important observation was:

```text
Condition TRUE  -> "Welcome back" message present
Condition FALSE -> "Welcome back" message absent
```

This response difference can be used as a **Boolean oracle** to test information from the database.

---

## Testing

### 1. Confirming the Boolean Response

A Boolean condition can first be tested using:

```sql
xyz' AND 1=1--
```

According to my notes, the response contains the `"Welcome back"` message when this condition is true.

The notes also mention that `OR` can be used, but the logical differences between `AND` and `OR` need to be understood before using them.

---

## 2. Checking Whether the Administrator User Exists

I tested whether a user named `administrator` exists in the `users` table:

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a'--
```

The `"Welcome back"` message was present.

This indicates that the subquery successfully found the `administrator` user and the comparison evaluated to true.

### False comparison

I then changed the comparison so that the selected character and comparison character were different:

```sql
xyz' AND (SELEDT 'a' FROM users WHERE username='administrator')='b'--
```

The notes record that the `"Welcome back"` message was not shown because the comparison does not match.

The important idea is that the selected value and the value being compared must satisfy the same Boolean condition for the true response to occur.

---

## 3. Determining the Password Length

After confirming that the `administrator` account exists, I tested the password length.

Example:

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>5)='a'--
```

The response contained the `"Welcome back"` message.

This means the condition:

```sql
LENGTH(password)>5
```

evaluated to true for the administrator password.

The reasoning is:

1. The query checks whether `administrator` exists.
2. It also checks whether the password length is greater than `5`.
3. If both conditions are true, the subquery returns `'a'`.
4. The outer comparison `'a'='a'` is true.
5. The application therefore produces the `"Welcome back"` response.

---

## 4. Narrowing Down the Password Length

I continued increasing the tested length until the response changed.

Example:

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGHT(password)>21)='a'--
```

The notes record that the response did **not** contain the `"Welcome back"` message.

This indicates that the tested length condition was false.

The notes then recorded a final equality check:

```sql
xyz' AND (SELECT 'a' FROM uses WHERE username='administrator' AND LENGHT(password)=20)='a'--
```

The response contained the `"Welcome back"` message, indicating that the password length was determined to be **20 characters**.

> **Note:** The payloads above are reproduced from my original notes. The notes contain spelling/typing variations such as `LENGHT` and `uses`; these are preserved here rather than silently changing the original lab record.

---

## 5. Finding the Password Character by Character

Once the password length was known, I used `SUBSTRING()` to test individual character positions.

The basic idea is:

```sql
SUBSTRING(password, position, character_count)
```

This allows one character of the password to be compared against a candidate character.

### First character

The first-position test from my notes was:

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND SUBSTRING(password,1,1)='x')='a'--
```

The `"Welcome back"` response indicated that the tested first character was correct according to the lab notes.

The process can then be repeated for each remaining position.

---

## 6. Testing the Second Character

For the second position, the payload can be changed to:

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND SUBSTRING(password,2,1)='x')='a'--
```

The position value changes from:

```text
1
```

to:

```text
2
```

The candidate character can then be changed repeatedly until the response indicates a match.

---

## 7. Alternative SUBSTRING Payload

My notes also contain an alternative form:

```sql
xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator')1,1)='x'--
```

The notes record that this form also worked.

The important concept is the same: retrieve the administrator password through a subquery and test individual characters against candidate values.

> **Note:** This payload is preserved from the original notes exactly. If reproducing it, verify the exact SQL syntax accepted by the lab database/application.

---

## Exploitation Process

The overall exploitation process was:

```text
Find Boolean response
        ↓
Confirm administrator exists
        ↓
Determine password length
        ↓
Test password position 1
        ↓
Test password position 2
        ↓
Continue through remaining positions
        ↓
Recover the complete password
        ↓
Log in as administrator
```

The key technique is not directly viewing the database response. Instead, information is recovered by repeatedly asking **true/false questions** and observing the application's response.

---

## Why It Worked

The application provides different responses depending on whether the injected SQL condition evaluates to true or false.

This creates a side channel that can be used to infer database information.

Conceptually:

```text
Injected condition
       ↓
   SQL evaluates
       ↓
 ┌─────┴─────┐
TRUE       FALSE
  ↓           ↓
"Welcome     No "Welcome
 back"        back"
```

Because the response reveals whether a condition is true, individual properties of the administrator account can be discovered without the application directly returning the database contents.

---

## Vulnerability

The application is vulnerable to **blind SQL injection using conditional responses**.

The vulnerability allows user-controlled input to influence SQL queries, while the application's response provides enough information to determine whether injected conditions are true or false.

---

## Impact

In this lab, the vulnerability can be used to:

* Determine whether the `administrator` account exists.
* Determine the administrator password length.
* Recover the password character by character.
* Obtain the administrator credentials.
* Log in as the administrator user.

In a real application, the impact would depend on the privileges associated with the compromised account and the information accessible through the vulnerable query.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements**.

Developers should:

* Use prepared statements for database queries.
* Keep SQL code separate from user-controlled data.
* Validate input appropriately.
* Apply least-privilege permissions to database accounts.
* Avoid exposing database errors or unnecessary response differences.
* Perform security testing against parameters that reach database queries.
* Implement appropriate authentication and authorization controls.

Input filtering alone should not be relied upon as the primary defense against SQL injection.

---

## Tools

* Burp Suite
* Web Browser
* Burp Repeater (for repeatedly testing Boolean conditions)

---

## Lessons Learned

* How blind SQL injection can be exploited through conditional responses.
* How a visible response difference can act as a Boolean oracle.
* How to verify whether a database user exists.
* How to determine a password length using `LENGTH()`.
* How `SUBSTRING()` can be used to test individual password characters.
* How information can be extracted without the application directly displaying database contents.
* Why parameterized queries are important for preventing SQL injection.
* How to document a blind SQL injection workflow clearly.

---

## Key Takeaway

This lab demonstrated that SQL injection does not require the application to directly display database results.

Even a small difference such as the presence or absence of a `"Welcome back"` message can reveal whether an injected SQL condition is true.

By repeatedly testing Boolean conditions, it is possible to infer information from the database one piece at a time.

The correct defense is to separate **SQL code from user-controlled data**, primarily through parameterized queries and prepared statements.
