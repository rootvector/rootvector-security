# Blind OS Command Injection with Time Delays

**Platform:** PortSwigger Web Security Academy
**Difficulty:** Practitioner
**Category:** OS Command Injection
**Status:** Solved

**Lab:** [Blind OS Command Injection with Time Delays](https://portswigger.net/academy/labs/launch/28bda028c1ba8408f4fccaaad3739fdc8d8e2d195f348c0db992cbcd6971cbf?referrer=%2fweb-security%2fos-command-injection%2flab-blind-time-delays)

---

## Objective

The application contains a **blind OS command injection** vulnerability in its feedback functionality.

The application executes a shell command containing user-supplied input, but the output of the command is not returned in the HTTP response.

The objective is to exploit the vulnerability and cause a **10-second delay** in the application's response.

---

## Initial Reconnaissance

The vulnerability is located in the feedback submission functionality.

The request contains an `email` parameter that is processed by the application.

Since the application does not return the output of the executed command, directly injecting a command and looking for its output would not work.

This makes it a **blind command injection** scenario.

---

## Testing

I intercepted the feedback request using **Burp Suite** and examined the parameters being submitted.

The `email` parameter was interesting because its value was processed by the server.

To test whether shell command injection was possible, I modified the parameter and used a command that would introduce a measurable delay.

---

## Payload

The `email` parameter was modified to:

```text
x||ping+-c+10+127.0.0.1||
```

The URL-encoded spaces represented by `+` allow the command to be passed through the HTTP parameter.

The important part of the payload is:

```text
ping -c 10 127.0.0.1
```

This sends 10 ICMP requests to the local host.

---

## Observation

After sending the modified request through Burp Suite, the server response took approximately **10 seconds** to return.

This behavior indicates that the injected command was executed by the server.

Because the command output was not reflected in the response, the delay provided a way to confirm command execution indirectly.

---

## Why It Worked

The application appears to construct and execute a shell command using the supplied `email` value.

The injected shell operators:

```text
||
```

allow another command to be executed after the application's original command.

Conceptually, the input changes the command execution flow from something like:

```text
original-command <user-input>
```

to a command sequence containing the injected command.

The `ping` command then introduces a predictable delay, allowing command execution to be detected without needing the command's output.

---

## Vulnerability

**Blind OS Command Injection**

The application passes user-controlled input into a shell command without properly preventing shell metacharacters or separating user input from command execution.

Unlike normal command injection, the result of the injected command is not directly visible in the HTTP response.

Time-based behavior can therefore be used as an indirect confirmation.

---

## Exploitation Flow

```text
Feedback functionality
        ↓
Identify user-controlled parameter
        ↓
Intercept request with Burp Suite
        ↓
Test the email parameter
        ↓
Inject shell command
        ↓
Application executes command
        ↓
No command output returned
        ↓
Use a time delay as an indicator
        ↓
Response delayed by ~10 seconds
        ↓
Blind OS command injection confirmed
```

---

## Impact

Blind OS command injection can allow an attacker to execute operating-system commands through a vulnerable application.

Depending on the privileges of the application process and the server environment, this can potentially lead to:

* Unauthorized command execution
* Access to sensitive files
* Modification or deletion of data
* Further compromise of the application or server
* Privilege escalation when combined with other weaknesses

The actual impact depends on the privileges available to the process executing the command.

---

## Remediation

Applications should avoid passing user-controlled input directly into a shell.

Recommended protections include:

* Avoid shell execution when an equivalent library/API is available.
* Do not concatenate user input into operating-system commands.
* Use safe APIs that pass arguments separately instead of invoking a shell.
* Apply strict allowlists where user input must control command arguments.
* Validate input according to its expected format.
* Run application processes with the minimum privileges required.
* Apply appropriate operating-system and application-level isolation.

Input filtering alone should not be treated as the primary defense against command injection.

---

## Tools Used

* Burp Suite
* PortSwigger Web Security Academy
* Web browser

---

## Key Takeaway

This lab demonstrated that command injection does not always require visible command output.

When an application executes injected commands but does not return their output, **observable side effects such as response timing can be used to confirm execution**.

The important lesson is to understand the underlying command execution flow rather than relying only on a specific payload.

