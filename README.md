# RootVector-Security

This repository contains my penetration testing lab work, security research, and notes from my cybersecurity learning journey.

I use this repository to document what I learn while working through web security labs, vulnerable applications, CTFs, and other authorized security environments.

The main goal is not just to complete a lab, but to understand why a vulnerability exists, how it can be exploited, what happens during exploitation, and how it can be prevented.

---

## What I Document

For each lab, I try to document the important parts of the process, including:

- Reconnaissance
- Enumeration
- Identifying the attack surface
- Testing different inputs and techniques
- Understanding the vulnerability
- Exploitation
- Impact
- Remediation
- Lessons learned

Some writeups may also include HTTP requests and responses, Burp Suite observations, commands, screenshots, and other useful evidence from the lab.

---

## Platforms

I currently use this repository to document labs from:

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [TryHackMe](https://tryhackme.com)
- CTF platforms
- Vulnerable web applications
- Local security labs
- Other authorized training environments

---

## Areas I'm Learning

### Web Security

- SQL Injection
- Authentication
- Access Control
- Cross-Site Scripting (XSS)
- Command Injection
- Path Traversal
- File Upload Vulnerabilities
- Server-Side Request Forgery (SSRF)
- Insecure Deserialization
- API Security

### Foundations

- Linux
- Networking
- TCP/IP
- HTTP
- DNS
- Bash
- C
- Python

### Penetration Testing

- Reconnaissance
- Enumeration
- Vulnerability Discovery
- Exploitation
- Privilege Escalation
- Post-Exploitation
- Security Tooling
- Automation

---

## How I Approach a Lab

I try to follow a simple process rather than jumping directly to an exploit:

```text
Understand the application
        ↓
Find the attack surface
        ↓
Test the input
        ↓
Observe the behavior
        ↓
Identify the vulnerability
        ↓
Understand why it works
        ↓
Exploit it in the lab
        ↓
Understand the impact
        ↓
Think about the fix
```

The most important part for me is understanding **why** something works.

For example, if I successfully exploit an SQL injection vulnerability, I don't want to remember only the payload. I want to understand how the input changed the SQL query and why the application allowed it.

---

## Repository Structure

```text
rootvector-labs/
│
├── PortSwigger/
│   ├── SQL-Injection/
│   ├── Authentication/
│   ├── Access-Control/
│   ├── Path-Traversal/
│   ├── Command-Injection/
│   ├── XSS/
│   └── ...
│
├── TryHackMe/
│   └── ...
│
├── Vulnerable-Applications/
│   ├── DVWA/
│   ├── bWAPP/
│   ├── WebGoat/
│   └── ...
│
└── README.md
```

The structure will change as I cover more topics and platforms.

---

## Tools

Depending on the lab, I use tools such as:

- Burp Suite
- Nmap
- curl
- Netcat
- ffuf
- Gobuster
- SQLMap
- Browser Developer Tools
- Python
- Bash
- Linux command-line tools

I try to understand what the tools are doing instead of treating them as black boxes.

---

## Learning Notes

This repository is also a reference for myself.

When I encounter something I don't understand, I document the explanation, commands, observations, and mistakes that helped me understand it.

Some writeups may therefore be more detailed than others. The repository represents my progress over time rather than a collection of perfect solutions.

---

## Scope and Ethics

Everything documented here is performed in intentionally vulnerable environments, CTFs, training platforms, local labs, or systems where I have permission to test.

The techniques and information in this repository should not be used against systems without authorization.

---

## Current Status

This repository is a work in progress.

I'm currently focusing on building a strong foundation in Linux, networking, programming, and web security while gradually moving deeper into penetration testing and offensive security.

More labs and writeups will be added as I continue learning.

---

## About Me

I'm a cybersecurity student interested in penetration testing, offensive security, Linux, networking, and low-level programming.

I use RootVector as my personal identity for documenting this journey and sharing the things I build and learn.

[GitHub](https://github.com/rootvector)
[TryHackMe](https://tryhackme.com/p/rootvector)
[Portfolio](https://rootvector.github.io/rootvector.sec/)
