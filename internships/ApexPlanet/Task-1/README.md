# Task 1 - FOUNDATIONS OF CYBERSECURITY

## Objective:

Build Strong fundamentals in cybersecurity, networking, cryptography, and set up a professional hacking lab.

---

## 1. Cybersecurity Basic

### Understand the CIA Traid: Confidentiality, Integrity, Availability.

The CIA Triad is one of the basic concepts in cybersecurity. It consists of three main principles: Confidentiality, Integrity, and Availability. These principles help us understand what security is trying to protect in a system.

#### Confidentiality

Confidentiality means keeping information private and allowing only authorized users to access it. For example, a user's password or personal information should not be visible to other users.

Access controls, strong passwords, and encryption are some ways to protect confidentiality.

#### Integrity

Integrity means keeping information accurate and preventing unauthorized changes. For example, if a web application stores a product price of ₹500, an unauthorized user should not be able to change it to ₹50.

Access permissions, input validation, and integrity checks can help protect data from unauthorized modification.

#### Availability

Availability means ensuring that systems and data are accessible when authorized users need them. For example, an online banking website should be available when customers need to use it.

Regular maintenance, backups, redundancy, and protection against denial-of-service attacks can help maintain availability.

#### My understanding

The CIA Triad covers three different security goals: keeping data private, keeping it correct, and keeping services accessible. A security issue can affect one or more of these principles at the same time.


### Explore Threat Types: Phishing, Malware, DDoS, SQL Injection, Brute Force, Ransomware.

During this section, I explored some common threats that can affect computers, networks, and web applications.

#### Phishing

Phishing is a technique in which an attacker uses a fake email, message, or website to trick someone into sharing sensitive information or performing an unsafe action.

For example, a fake login page may look like a legitimate service and ask users to enter their credentials.

Prevention: Verify links and sender addresses, use multifactor authentication, and avoid entering credentials on suspicious websites.

#### Malware

Malware is software designed to perform harmful or unauthorized activities on a computer or network. Viruses, worms, Trojans, spyware, and ransomware are examples of malware.

Malware may steal information, damage files, or allow unauthorized access to a system.

Prevention: Keep software updated, use trusted software sources, and avoid opening suspicious attachments.

#### Distributed Denial of Service (DDoS)

A DDoS attack attempts to make a service unavailable by overwhelming it with traffic or requests from multiple systems.

For example, a website may become slow or unavailable because it is receiving more requests than it can handle.

Prevention: Traffic filtering, rate limiting, monitoring, and DDoS protection services can help reduce the impact.

#### SQL Injection

SQL Injection (SQLi) is a web application vulnerability that occurs when untrusted input is incorrectly included in a database query. Depending on the vulnerability and database permissions, an attacker may be able to read or modify data or bypass application controls.

Prevention: Use parameterized queries, validate input appropriately, and apply least-privilege permissions to database accounts.

#### Brute Force

A brute-force attack involves repeatedly trying possible passwords or credentials until a valid combination is found. Weak and reused passwords make these attacks easier.

Prevention: Use unique passwords, multifactor authentication, rate limiting, and appropriate account lockout or delay mechanisms.

#### Ransomware

Ransomware is a type of malware that commonly encrypts files or otherwise prevents access to data and demands payment. Some attackers also threaten to publish stolen information.

Prevention: Maintain tested offline or isolated backups, patch systems, restrict permissions, and train users to recognize suspicious messages and files.


### Study Attack Vectors: Social Engineering, Wireless Attacks, Insider Threats.

An attack vector is a path or method an attacker may use to reach a system, exploit a weakness, or gain unauthorized access.

#### Social Engineering

Social engineering involves manipulating people into revealing information, granting access, or performing actions that weaken security.

For example, someone may impersonate technical support and ask an employee for their password.

Prevention: Verify unusual requests independently, never share passwords, and provide security-awareness training.

#### Wireless Attacks

Wireless attacks target weaknesses in Wi-Fi networks, wireless protocols, configurations, or connected devices.

Examples include attempting to exploit weak Wi-Fi passwords, using rogue access points, or intercepting traffic on an inadequately protected network.

Prevention: Use modern Wi-Fi security such as WPA2-AES or WPA3, choose strong passwords, update router firmware, and avoid trusting unknown access points.

#### Insider Threats

An insider threat comes from someone who already has legitimate access to an organization's systems or information. This could involve intentional misuse or an accidental mistake.

For example, an employee might expose confidential files by sharing them with the wrong people.

Prevention: Apply least privilege, review access permissions, monitor important activity appropriately, and train staff to handle information securely.


### Key Takeaways

-    The CIA Triad helps define the main goals of information security.

-    Cybersecurity threats can target people, applications, devices, and networks.

-    An attack vector is the path used to attempt an attack, while a threat is a potential cause of harm.

-    Security requires multiple layers of protection rather than relying on a single tool.

-    Understanding these fundamentals is important before moving into practical security testing.

### References

- [CISA — Secure Our World](https://www.cisa.gov/secure-our-world)

- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

- [pOWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

---

## 2. Lab Environment Setup

### Install VirtualBox or VMware.

First install the required softwares, The VirtualBox or VMware provides a virtual environment. We are use VirtualBox in this session to run a attacker machine (Kali Linux 2026.2) and target machine (Metasploitable 2).

> Download Links:

[VirtualBox](https://www.virtualbox.org/)


The Video is also available at
[Setting Up a Haking Lab]()

### Install Kali Linux as the attacker machine.

Kali Linux is a open source linux distibution based on debian. Design for penetration testing, ethical hacking, and for cyber security experst.

[Kali Linux](https://www.kali.org/get-kali/#kali-platforms)


### Install Metasploitable 2 or DVWA (Damn Vulnerable Web App) as target machines.

Metasploitable 2 is a vulnerable machine design to learn ethical hacking safely and exploit the machine without having a any issue. 

[Metasploitable 2](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)


### Configure a private lab network (Host-Only Adapter)

Setting up the Host-Only Adapter on both machine help to communicate between those thow mahines without interacting with your host machine in safe virtual local area network (VLAN).

---

## 3. Linux Fundamentals
