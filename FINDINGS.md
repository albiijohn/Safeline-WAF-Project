# Findings: SafeLine WAF and DVWA Lab

## Summary

This file summarizes the main security controls configured and observed during the SafeLine WAF and DVWA homelab.

The purpose of this file is to document what was tested, what was observed, and what security value each control provided.

---

## Finding 1: SafeLine Was Placed in Front of DVWA

| Field          | Details                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------ |
| Control        | Reverse proxy protection                                                                         |
| Observation    | DVWA traffic was routed through SafeLine before reaching the Ubuntu web server                   |
| Security Value | Allows SafeLine to inspect and control traffic before it reaches the vulnerable application      |
| Recommendation | Keep application traffic routed through the WAF instead of exposing the backend service directly |

---

## Finding 2: HTTPS Access Was Configured

| Field          | Details                                                                                        |
| -------------- | ---------------------------------------------------------------------------------------------- |
| Control        | SSL/TLS configuration                                                                          |
| Observation    | A self-signed certificate was created and imported into SafeLine                               |
| Security Value | Demonstrates encrypted access to the protected application                                     |
| Recommendation | Use trusted certificates in production environments; use self-signed certificates only in labs |

---

## Finding 3: HTTP Flood Protection Was Tested

| Field          | Details                                                                                            |
| -------------- | -------------------------------------------------------------------------------------------------- |
| Control        | Rate limiting                                                                                      |
| Observation    | SafeLine was configured to block repeated requests from the test machine after a defined threshold |
| Security Value | Helps reduce the impact of repeated automated or abusive requests                                  |
| Recommendation | Tune thresholds carefully to avoid blocking normal users while still reducing abusive traffic      |

---

## Finding 4: Authentication Was Added Before Application Access

| Field          | Details                                                                               |
| -------------- | ------------------------------------------------------------------------------------- |
| Control        | SafeLine authentication rule                                                          |
| Observation    | An authentication page was enabled before accessing the protected application         |
| Security Value | Adds an extra access-control layer before users reach DVWA                            |
| Recommendation | Use strong credentials and avoid weak default test passwords outside lab environments |

---

## Finding 5: Custom IP Blocking Was Configured

| Field          | Details                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------ |
| Control        | IP-based deny rule                                                                               |
| Observation    | A rule was created to block access from the Kali test machine IP                                 |
| Security Value | Demonstrates how a WAF can restrict known unwanted sources                                       |
| Recommendation | Use IP blocking carefully; combine it with logging and review to avoid blocking legitimate users |

---

## Finding 6: SQL Injection Protection Was Observed

| Field          | Details                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| Control        | WAF attack protection rule                                                                             |
| Observation    | SafeLine blocked a basic SQL injection attempt against DVWA when protection rules were enabled         |
| Security Value | Shows how a WAF can help detect and block common web attack patterns                                   |
| Recommendation | Do not rely only on WAF protection; fix the vulnerable application code and validate input server-side |

---

## Overall Result

The lab showed that SafeLine can be used to protect a deliberately vulnerable web application by adding traffic inspection, access control, rate limiting, custom blocking, HTTPS support, and attack protection.

This project helped me understand how WAF controls are deployed and validated in a controlled lab environment.

---

## Important Note

The original VM environment used for testing is no longer available. These findings are based on the original setup documentation and screenshots captured during the lab.

No new screenshots or test results are added in this file.

---

## Recommended Next Steps

If this lab is rebuilt later, I would add:

* Fresh SafeLine event log screenshots
* SQL injection blocked-request evidence
* Before/after test comparison
* Rule tuning notes
* Short remediation summary for each finding
* A clean architecture diagram
