# Security First Principles: Least Privilege
Sample learning module for a Dakota State University MS-level computer science (CSC) course

[Least Privilege Lecture](https://youtu.be/hmzqLS8m448)

## General Information
-   Author: Wyatt Tauber
-   Date: November 27th, 2023
-   Description: This course module discusses the security first principle of least privilege as well as privilege escalation vulnerabilities.

## Why You Should Care
Access control rules apply to every object in an operating system. Organizations frequently experience data loss through privilege escalation vulnerabilities resulting from the misconfiguration of this access for services or user accounts. Some of the most common problems result from internet-facing features running as root, employees with significant scope creep as they change roles, and the misconfiguration of program or file permissions. Many other privilege escalation vulnerabilities are far more nuanced.

Each of these issues can be addressed by the security first principle of least privilege, which ensures that users are given only the minimum level of privileges and permissions needed to perform required tasks. Organizations can protect themselves through the implementation of standardized security models, technical safeguards, and routine job responsibility audits. These actions reduce an organization’s attack surface, significantly curtailing the risk to business assets, operations, and people.

## Three Main Ideas
1. A privilege is the ability of a user or service to act on managed computer resources.
    -   Privileges are typically implemented on users and processes.
    -   Privileges affect access to and the ability to act on files and other resources.
2. The principle of least privilege supports security first design by minimizing the number of privileges granted to a user for accomplishing assigned duties and disabling privileges that are unused or not required for accomplishing assigned duties.
    -   Access control methods are formal presentations of the security policies enforced by the operating system.
    -   Security models align with access control methods to enforce policies, supporting the principle of least privilege.
3. Horizontal and vertical privilege escalation (PrivEsc) vulnerabilities exploit configuration errors to gain unauthorized access into a system and access additional privileges or resources when the principle is not followed.

## Example
### Scenario
An overworked systems administrator for a small business is deploying a LAMP (Linux, Apache, MySQL, PHP) stack on a CentOS box in the company's DMZ to test a new e-commerce platform before its launch. The administrator is encountering several conflicts with SELinux (Security Enhanced Linux), which is blocking new PHP connections to the MySQL database and preventing Apache from accessing files in the `/var/www` directory.

### Poor Practice
The administrator doesn't want to spend time configuring and debugging SELinux policies since this is just a test server. Therefore, the administrator disables SELinux and runs the LAMP stack as the root user to avoid further issues. This violates the principle of least privilege as web services do not need to access to the entire operating system to function.

A new CVE for this version of Apache is announced a few days later, and the administrator doesn't have time to pay attention to or patch these either. The unpatched server shows up in an attacker's routine `nmap` scans, and they compromise the Apache web server and deface the e-commerce platform to sell counterfeit fashion products for several weeks. The small business becomes aware of the issue when they receive a cease and desist letter from the fashion company for distributing counterfeit products, and their reputation is damaged.

### Best Practice
The administrator spends a significant amount of time learning how to set up LAMP securely on CentOS, and how to write proper SELinux rules for a web server. Management isn't terribly happy about the time it took to set up the server, but the administrator's completed SELinux configuration ends up being deployed on the production webserver as well.

Even though the administrator still don't have enough time to patch the Apache CVE, the attacker is relatively unskilled and unwilling to spend time attempting to exploit a target with SELinux running. They move on to an easier target. When several competitiors are compromised and their websites are taken down, the small business gains several new customers.

## Additional Resources
### Privileges and Permissions
[Introduction to Cybersecurity First Principles - Least Privilege](https://mlhale.github.io/nebraska-gencyber-modules/intro_to_first_principles/README/)

#### Windows
[Hidden Danger: How To Identify and Mitigate Insecure Windows Services](https://offsec.blog/hidden-danger-how-to-identify-and-mitigate-insecure-windows-services/)

#### Linux
[Linux permissions: SUID, SGID, and sticky bit](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit)

[Chapter 12. Managing sudo access](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/managing-sudo-access_configuring-basic-system-settings)

### Security Models
[Introduction To Classic Security Models](https://www.geeksforgeeks.org/introduction-to-classic-security-models/)

### Access Control
[Verification and Test Methods for Access Control Policies/Models](https://csrc.nist.gov/pubs/sp/800/192/final)

[Comparing Access Control: RBAC, MAC, DAC, RuBAC, ABAC](https://techgenix.com/5-access-control-types-comparison/)

### Privilege Escalation
[What Is Privilege Escalation? - ProofPoint](https://www.proofpoint.com/us/threat-reference/privilege-escalation)

[What is Privilege Escalation? - CrowdStrike](https://www.crowdstrike.com/cybersecurity-101/privilege-escalation/)

[Understanding Privilege Escalation and 5 Common Attack Techniques](https://www.cynet.com/network-attacks/privilege-escalation/)

#### Windows
[Privilege escalation on Windows: When you want it and when you don’t](https://delinea.com/blog/windows-privilege-escalation)

#### Linux
[A Guide On Linux Privilege Escalation - The Journey Towards System Control](https://iasad.me/blogs/linux-privilege-escalation/)
