# OS Command Injection, Simple Case

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Apprentice
**Category:** OS Command Injection
**Status:** Solved

**Lab:** [OS Command Injection, Simple Case](https://portswigger.net/academy/labs/launch/2b08de150e353dc1c0057519f466a55a0ca4a7901b2469d56ef7d2a80eddeaa6?referrer=%2fweb-security%2fos-command-injection%2flab-simple)

---

## Objective

The application contains an **OS command injection** vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs and returns the raw output of the command in the HTTP response.

The objective is to execute the `whoami` command and determine the name of the current user.

---

## Initial Reconnaissance

The vulnerable functionality is the **product stock checker**.

The stock checker sends a request containing product and store identifiers. Since these values are processed by the server and the response contains command output, the parameters are worth testing for OS command injection.

The `storeID` parameter was identified as an interesting input.

---

## Testing

I intercepted the stock checker request using **Burp Suite** and examined the parameters.

The `storeID` parameter was modified to test whether additional shell commands could be executed.

The following value was used:

```text
1|whoami
```

---

## Payload

```text
1|whoami
```

The `|` character is a shell pipe operator.

It allows the output of the command before it to be passed to the command after it.

In this case, the injected `whoami` command is executed by the operating system.

---

## Observation

After sending the modified request, the HTTP response contained the output of the injected command.

The response revealed the name of the user under which the application was running.

This confirmed that the application was executing the injected operating-system command.

---

## Why It Worked

The application uses user-controlled input as part of a shell command without properly separating the input from command execution.

Conceptually, the application executes something similar to:

```text
original-command <productID> <storeID>
```

By injecting:

```text
1|whoami
```

the shell interprets the `|` as a command separator and executes `whoami`.

Because the application returns the command's output in the response, the result of `whoami` can be directly observed.

---

## Vulnerability

**OS Command Injection**

The vulnerability occurs because user-controlled input is incorporated into an operating-system command executed by the server.

An attacker can potentially manipulate the command and execute additional operating-system commands with the privileges of the vulnerable application.

---

## Exploitation Flow

```text
Product stock checker
        ↓
Identify user-controlled parameter
        ↓
Intercept request with Burp Suite
        ↓
Test storeID parameter
        ↓
Inject shell command
        ↓
Application executes injected command
        ↓
Command output returned
        ↓
whoami output observed
        ↓
OS command injection confirmed
```

---

## Impact

OS command injection can allow an attacker to execute arbitrary operating-system commands on the server.

Depending on the privileges of the application process, this can potentially result in:

* Reading sensitive files
* Modifying application data
* Executing additional commands
* Accessing system information
* Further compromise of the server
* Privilege escalation when combined with other vulnerabilities

The actual impact depends on the permissions available to the application process.

---

## Remediation

Applications should avoid passing user-controlled input directly into shell commands.

Recommended protections include:

* Avoid shell execution when an equivalent library or API is available.
* Do not concatenate user input into operating-system commands.
* Use APIs that pass arguments separately instead of invoking a shell.
* Apply strict allowlists for expected parameter values.
* Validate input according to its expected format.
* Run application processes with the minimum privileges required.

Input filtering should not be considered the primary defense against command injection.

---

## Tools Used

* Burp Suite
* PortSwigger Web Security Academy
* Web browser

---

## Key Takeaway

This lab demonstrated a basic OS command injection vulnerability where user-controlled input was directly incorporated into a shell command.

The important lesson is that parameters that appear to contain simple values such as IDs should still be tested to determine whether they reach a shell command.

In this case, the command output was returned directly in the response, making the injected command and its result easy to observe.
