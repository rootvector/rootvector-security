# Blind OS Command Injection with Output Redirection

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Practitioner
**Category:** OS Command Injection
**Status:** Solved

**Lab:** [Blind OS Command Injection with Output Redirection](https://portswigger.net/academy/labs/launch/794c7ab29027c16b9a3fab84326c6f313a8af27a96f1833828edd38b26a4aca7?referrer=%2fweb-security%2fos-command-injection%2flab-blind-output-redirection)

---

## Objective

This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command using user-supplied input, but the output of the command is not directly returned in the response.

The goal is to execute the `whoami` command and retrieve its output.

---

## Initial Reconnaissance

I started by checking the feedback functionality and intercepted the request using **Burp Suite**.

The `email` parameter looked interesting because the application uses the submitted details while processing the feedback.

Since this is a blind command injection, I couldn't directly see the output of an injected command in the response.

The lab also provides a writable directory:

```text
/var/www/images/
```

The application uses this directory to serve product images, so I can use it to store the output of the injected command.

---

## Testing

I intercepted the feedback request in Burp Suite and modified the `email` parameter.

I used output redirection to write the result of `whoami` into a file inside the writable images directory.

Payload:

```text
||whoami>/var/www/images/output.txt||
```

So the complete parameter became:

```text
email=||whoami>/var/www/images/output.txt||
```

---

## What Happened

The `>` operator redirects the output of a command into a file.

In this case:

```text
whoami > /var/www/images/output.txt
```

means that instead of returning the output directly, the result of `whoami` is written to:

```text
/var/www/images/output.txt
```

The application serves files from this directory, so I could later request the file through the image loading functionality.

---

## Retrieving the Output

After submitting the modified feedback request, I intercepted a request that loads a product image.

The request contained a `filename` parameter.

I changed the parameter to:

```text
filename=output.txt
```

This caused the application to return the contents of the file created by the injected command.

The response contained the output of:

```text
whoami
```

This confirmed that the command had been executed successfully.

---

## Exploitation Flow

```text
Feedback function
        ↓
Find user-controlled email parameter
        ↓
Intercept request with Burp Suite
        ↓
Inject whoami command
        ↓
Redirect output to output.txt
        ↓
File written to /var/www/images/
        ↓
Find image loading request
        ↓
Change filename to output.txt
        ↓
Retrieve command output
```

---

## Why It Worked

The application passes user-controlled input into a shell command.

The injected shell operators allow another command to be executed:

```text
||
```

The `whoami` command then gets executed and its output is redirected using:

```text
>
```

to a file inside the writable directory:

```text
/var/www/images/output.txt
```

Because the application also allows files from this directory to be requested through the image functionality, the output can be retrieved from the created file.

This is why the vulnerability can still be exploited even though the command output isn't directly included in the feedback response.

---

## Vulnerability

**Blind OS Command Injection with Output Redirection**

The application allows user-controlled input to reach a shell command.

Even though the output is not directly reflected in the HTTP response, an attacker can redirect the command output to a location that is accessible through another part of the application.

---

## Impact

OS command injection can potentially allow an attacker to execute commands on the server with the privileges of the application process.

Depending on the environment and permissions, this could lead to:

* Reading sensitive files
* Modifying files
* Executing additional commands
* Accessing system information
* Further compromise of the application or server
* Privilege escalation when combined with other vulnerabilities

The actual impact depends on the privileges of the process running the vulnerable application.

---

## Remediation

The application should avoid passing user-controlled input directly into shell commands.

Some recommended protections are:

* Avoid shell commands when a safer API or library can be used.
* Never concatenate user input directly into commands.
* Pass arguments separately instead of invoking a shell where possible.
* Validate input against a strict allowlist.
* Run the application with minimum required privileges.
* Restrict write access to application directories.
* Prevent user-controlled input from determining filesystem paths.

---

## Tools Used

* Burp Suite
* PortSwigger Web Security Academy
* Web browser

---

## Key Takeaway

This lab showed me how blind OS command injection can still be exploited when the command output is not directly visible.

Instead of relying on the response, I redirected the output of `whoami` into a writable file and then used another application feature to retrieve that file.

The main thing I learned from this lab is that **blind command injection doesn't mean command execution cannot be confirmed**. Other application functionality can sometimes be used to observe the result indirectly.
