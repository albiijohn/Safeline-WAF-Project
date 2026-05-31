# Case Study: SafeLine WAF Protection Lab for DVWA

## Executive Summary

This project documents a web application security lab where I deployed **SafeLine Web Application Firewall (WAF)** in front of **Damn Vulnerable Web Application (DVWA)** to understand how a WAF can help protect a vulnerable web application.

The lab was built using Kali Linux and Ubuntu Server in VMware. DVWA was hosted on Ubuntu, and SafeLine was configured as a reverse proxy to inspect and control traffic before it reached the application.

This project helped me understand practical WAF deployment, HTTPS configuration, access control, rate limiting, and basic web attack mitigation.

---

## Objective

The objective of this lab was to:

* Deploy DVWA as a vulnerable test application.
* Install and configure SafeLine WAF.
* Place SafeLine in front of DVWA as a reverse proxy.
* Configure HTTPS access using a self-signed certificate.
* Test basic access control and rate limiting.
* Observe how SafeLine handles common web attack attempts such as SQL injection.
* Document the configuration and security controls used in the lab.

---

## Lab Environment

| Component                | Details                 |
| ------------------------ | ----------------------- |
| Attacker/Test Machine    | Kali Linux              |
| Web Server               | Ubuntu Server           |
| Vulnerable Application   | DVWA                    |
| Web Application Firewall | SafeLine WAF            |
| Virtualization           | VMware                  |
| Network Type             | Local lab network       |
| Test Scope               | Authorized homelab only |

---

## Architecture

```text
+------------------+
| Kali Linux       |
| Test Browser     |
| Security Testing |
+--------+---------+
         |
         v
+------------------+
| SafeLine WAF     |
| HTTPS / Rules    |
| Reverse Proxy    |
+--------+---------+
         |
         v
+------------------+
| Ubuntu Server    |
| Apache + DVWA    |
| Port 8080        |
+------------------+
```

---

## Implementation Summary

| Phase   | Activity                                                                               |
| ------- | -------------------------------------------------------------------------------------- |
| Phase 1 | Installed Kali Linux and Ubuntu Server in VMware                                       |
| Phase 2 | Deployed DVWA on Ubuntu Server                                                         |
| Phase 3 | Changed DVWA/Apache service to run behind SafeLine                                     |
| Phase 4 | Installed and accessed SafeLine WAF dashboard                                          |
| Phase 5 | Added DVWA as a protected application                                                  |
| Phase 6 | Configured HTTPS with a self-signed certificate                                        |
| Phase 7 | Tested rate limiting, authentication, custom IP blocking, and SQL injection protection |

---

## Controls Tested

| Control                  | Purpose                                                      |
| ------------------------ | ------------------------------------------------------------ |
| Reverse Proxy            | Forward traffic through SafeLine before reaching DVWA        |
| HTTPS Access             | Provide encrypted access to the protected application        |
| HTTP Flood Protection    | Limit repeated requests from a source IP                     |
| Authentication Rule      | Add an authentication layer before accessing the application |
| Custom IP Blocking       | Block traffic from a specific test IP                        |
| SQL Injection Protection | Detect and block basic SQL injection attempts                |
| Dashboard Monitoring     | Review traffic, blocks, and statistics                       |

---

## Key Observations

SafeLine provided a central place to configure application protection rules and review security-related activity. The lab showed how a WAF can sit between a user and a vulnerable application to enforce access controls and block certain attack patterns.

The most useful part of this project was understanding the workflow:

1. Host a vulnerable application.
2. Place a WAF in front of it.
3. Configure application routing.
4. Enable protection features.
5. Test from a controlled machine.
6. Review dashboard results.
7. Document what was blocked and why.

---

## Skills Demonstrated

* Web application security lab setup
* WAF deployment and configuration
* DVWA configuration
* Apache and PHP troubleshooting
* Reverse proxy setup
* HTTPS certificate configuration
* Rate limiting validation
* Access control testing
* Basic SQL injection protection testing
* Security documentation

---

## Limitations

The original VM environment used for this lab is no longer available, so this case study is based on the setup notes and screenshots captured during the original implementation.

This project should be treated as a documented homelab implementation, not a production security assessment.

---

## Future Improvements

If rebuilt, I would improve the project by adding:

* Fresh screenshots of blocked requests
* Before-and-after SQL injection test evidence
* SafeLine event log screenshots
* A short attack/control matrix
* More detailed remediation notes
* Rule tuning examples
* A cleaner network diagram

---

## Role Relevance

This project is relevant to:

* Security Support Analyst
* Junior Cybersecurity Analyst
* Vulnerability Management Analyst
* Web Application Security Support
* SOC Analyst Level 1
* Technical Support Analyst with security responsibilities
