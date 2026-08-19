# Gap-analysis-system-config-
Security configuration gap analysis — comparing a Microsoft security baseline against live audit policy settings using Policy Analyzer, with severity-ranked findings and remediation plan.


System Configuration Gap Analysis — Windows Audit Policy Baseline
Overview
I put together this lab project while working through CompTIA Security+ training. The goal was to get hands-on experience with security configuration management — specifically, comparing a recommended Microsoft security baseline against the actual (“effective”) settings running on a live Windows machine, and identifying where the two don’t match. This is a core part of how real organizations catch configuration drift and unmonitored systems before they become incidents.
What This Lab Is About
Basically, I took a Microsoft-published security baseline (a known-good set of audit policy settings) and ran it against the live configuration of a Windows 10 target machine (PC10) using the Policy Analyzer tool. The idea is simple but important: baselines only protect you if the system is actually configured to match them. This lab walks through finding the gaps between “what should be configured” and “what’s actually configured.”
What I Wanted to Learn
	•	How to use Microsoft’s Policy Analyzer to compare security baselines against a live system
	•	What “effective state” means vs. a documented baseline
	•	How to read and interpret Windows Advanced Audit Policy settings
	•	How to prioritize configuration gaps by risk/severity
	•	How to write up findings in a way a security team could actually act on
	•	General PowerShell navigation for locating lab/reference files
  
Tools I Used:
|Tool                                                   |What It Does                                                       |Why I Used It                                    |
|-------------------------------------------------------|-------------------------------------------------------------------|-------------------------------------------------|
|**Windows 10 (PC10)**                                  |Target machine being assessed                                      |The system whose real-world config I was auditing|
|**Microsoft Policy Analyzer v4.0**                     |Compares baseline `.PolicyRules` files against live effective state|Core tool for this whole analysis                |
|**Windows PowerShell ISE**                             |Script editor / terminal                                           |Navigated lab files, ran directory listings      |
|**Microsoft Security Baselines** (Win10 v1809 + WS2019)|Pre-built recommended policy sets                                  |Used as the “desired state” reference            |

How I Worked Through This
Step 1: Located the Lab Files
	•	Ran PowerShell as Administrator on PC10
	•	Copied lab reference material from the mounted drive (copy D:\* c:\LABFILES)
	•	Browsed the resulting folder structure, which included baseline policy sets, sample logs, vulnerability scan data (NVD-Control-RA-5-VULNERABILITY), and incident response templates (CSIRT Incident Handling Form)
Step 2: Loaded the Security Baseline
	•	Opened Policy Analyzer and pointed it at the Microsoft-published baseline set: MSFT-Win10-v1809-RS5-WS2019-FINAL
	•	This baseline represents Microsoft’s recommended audit policy configuration for a Windows 10 (RS5/1809) system in a Windows Server 2019 domain environment
Step 3: Captured the Effective State
	•	Used Policy Analyzer’s “Compare to Effective State” function to pull the actual, currently-applied audit policy settings from PC10
	•	This produced a live snapshot (EffectiveState_PC10_20260709...) representing what’s really configured on the machine, regardless of what the baseline recommends
Step 4: Ran the Comparison
	•	Opened the Policy Viewer (393 policy items total) showing baseline vs. effective state side by side
	•	Rows where the two didn’t match were automatically highlighted in yellow by the tool
Step 5: Documented the Gaps
	•	Went through the highlighted rows and recorded each mismatch
	•	Categorized them by what kind of gap it represented (missing auditing entirely vs. partial auditing vs. auditing the wrong event type)
	•	Assigned rough severity based on what the setting is meant to protect against
Executive Summary
Out of 393 audit policy settings compared, a meaningful subset of subcategories showed a mismatch between the Microsoft-recommended baseline and PC10’s actual effective configuration. The most common pattern was auditing being recommended but not actually enabled (“No Auditing” in the effective column) — meaning that if a relevant security event occurred in one of those categories, it would go completely unlogged. A smaller number of settings were partially configured (auditing “Success” only, when the baseline calls for “Success and Failure,” or vice versa), which limits visibility into failed/malicious attempts specifically.
Taken together, these gaps mean PC10 currently has reduced audit visibility in several high-value areas — including removable storage use, file share access, directory service changes, and process creation — that would matter most during an actual incident investigation.

Gap Identification & Severity Levels
|# |Policy Setting                 |Gap Type                                             |Severity|
|--|-------------------------------|-----------------------------------------------------|--------|
|1 |Removable Storage              |Not audited at all                                   |🔴 High  |
|2 |Directory Service Changes      |Not audited at all                                   |🔴 High  |
|3 |Account Lockout                |Wrong event type audited (Success instead of Failure)|🔴 High  |
|4 |Process Creation               |Not audited at all                                   |🔴 High  |
|5 |MPSSVC Rule-Level Policy Change|Not audited at all                                   |🔴 High  |
|6 |Other Logon/Logoff Events      |Not audited at all                                   |🟠 Medium|
|7 |File Share                     |Not audited at all                                   |🟠 Medium|
|8 |Other Object Access Events     |Not audited at all                                   |🟠 Medium|
|9 |Other Account Management Events|Not audited at all                                   |🟠 Medium|
|10|User Account Management        |Partially audited (missing Failure)                  |🟠 Medium|
|11|PNP Activity                   |Not audited at all                                   |🟠 Medium|
|12|Group Membership               |Not audited at all                                   |🟠 Medium|
|13|Directory Service Access       |Partially audited (missing Failure)                  |🟡 Low   |
|14|Detailed File Share            |Not audited at all                                   |🟡 Low   |
|15|Credential Validation          |Partially audited (missing Failure)                  |🟡 Low   |

Settings that matched the baseline (no gap): Computer Account Management, Security Group Management, Logon, Special Logon, Audit Policy Change, Authentication Policy Change.

Current vs. Desired State
|Policy Setting                         |Desired State (Baseline)|Current State (Effective)|Risk If Left Unaddressed                                                                 |
|---------------------------------------|------------------------|-------------------------|-----------------------------------------------------------------------------------------|
|Removable Storage                      |Success and Failure     |No Auditing              |USB/external media use goes completely untracked — a common data exfiltration path       |
|Directory Service Changes              |Success and Failure     |No Auditing              |Changes to AD objects (users, groups, permissions) leave no trail                        |
|Account Lockout                        |Failure                 |Success                  |Failed logon attempts that trigger lockouts (a brute-force indicator) aren’t being logged|
|Process Creation                       |Success                 |No Auditing              |No record of what processes/executables actually ran on the machine                      |
|MPSSVC Rule-Level Policy Change        |Success and Failure     |No Auditing              |Firewall rule changes (e.g., opening a port for malware C2) go unrecorded                |
|File Share / Other Object Access Events|Success and Failure     |No Auditing              |No visibility into who accessed shared files or other protected objects                  |
|User Account Management                |Success and Failure     |Success only             |Failed attempts to create/modify accounts (privilege escalation attempts) aren’t logged  |
|PNP Activity                           |Success                 |No Auditing              |New hardware/device connections aren’t tracked                                           |

Implementation Priority
|Priority       |Action                                                                                                                                                |Timeframe     |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
|P1 (Immediate) |Enable auditing for Removable Storage, Process Creation, Directory Service Changes, MPSSVC Rule-Level Policy Change                                   |Within 1 week |
|P1 (Immediate) |Correct Account Lockout audit setting (Success → Failure)                                                                                             |Within 1 week |
|P2 (Short-term)|Enable auditing for File Share, Other Object Access Events, Other Logon/Logoff Events, Other Account Management Events, Group Membership, PNP Activity|Within 30 days|
|P3 (Planned)   |Complete partial-audit categories (User Account Management, Directory Service Access, Credential Validation)                                          |Within 60 days|
|P4 (Ongoing)   |Re-baseline and re-compare on a recurring schedule (e.g., quarterly) to catch new drift                                                               |Recurring     |

How This Connects to CompTIA Security+
This lab touches on several CompTIA Security+ domains:
Domain 1.0: Threats, Attacks, and Vulnerabilities
	•	Understanding what audit blind spots mean for detecting real attacks
Domain 2.0: Architecture, Design, and Implementation
	•	Security baseline application and configuration management
Domain 4.0: Operations and Incident Response
	•	The direct link between audit policy and incident investigation capability — you can’t investigate what you never logged
Domain 5.0: Governance, Risk, and Compliance
	•	Gap analysis against a published baseline is a standard GRC/compliance exercise
  
What’s in This Repo
system-config-gap-analysis/
├── README.md                              # This file
├── screenshots/
│   ├── 01-policy-analyzer-baseline-list.png
│   ├── 02-effective-state-comparison.png
│   ├── 03-policy-viewer-full-results.png
│   ├── 04-labfiles-directory-listing.png
│   ├── 05-network-config-pc10.png
│   └── 06-powershell-admin-session.png

Skills I Built
✅ Running a security baseline comparison with Policy Analyzer
✅ Reading Windows Advanced Audit Policy configuration
✅ Identifying and categorizing configuration gaps
✅ Assigning risk-based severity to findings
✅ Writing a gap analysis report a real team could act on
✅ Basic PowerShell navigation and file management
✅ Connecting technical findings back to compliance/governance concepts
What’s Next for Me
I’m not stopping here. I want to:
	•	Actually push a corrected audit policy via Group Policy and re-run the comparison to confirm remediation
	•	Learn how to pipe these audit events into a SIEM and build detection rules around them
	•	Get my CompTIA Security+ cert — working on it now
	•	Explore CIS Benchmarks as another baseline source, not just Microsoft’s
	•	Keep documenting labs like this as I go
How to Look at This Repo
	•	Start with this README for the full analysis
	•	Check out the screenshots to see the Policy Analyzer comparison directly
	•	Feel free to ask questions — I’m still learning too
What I Learned
	•	A security baseline is only useful if you actually check systems against it — writing the policy isn’t the same as enforcing it
	•	“No Auditing” isn’t a neutral gap, it’s a blind spot — if it’s not logged, it didn’t happen as far as your evidence is concerned
	•	Partial auditing (Success only, missing Failure) is an easy gap to miss because something is being logged, just not the part that usually matters most for detecting an attack
	•	Prioritization matters — not every gap is equally urgent, and a good gap analysis ranks them instead of listing them flat
	•	Configuration drift is likely to affect more than one machine, so fixes belong at the policy/GPO level, not the individual endpoint
Important Note
⚠️ Real talk:
	•	This lab was performed in an isolated, instructor-provided lab environment (CompTIA’s assisted lab platform) on a lab-only machine (PC10)
	•	No production systems were assessed or modified
	•	IP addresses, hostnames, and other identifiers shown are lab environment values, not real infrastructure
Resources I Used
	•	Microsoft Security Compliance Toolkit / Policy Analyzer — baseline comparison tool
	•	Microsoft Security Baselines — reference baselines
	•	Windows Advanced Audit Policy documentation — reference material
	•	CompTIA Security+ — studying for this
About Me
Samuel donkor
Security-focused learner | CompTIA Security+ in progress | Configuration & Compliance enthusiast
I’m genuinely interested in cybersecurity and want to make it my career. I’m the kind of person who digs into problems to understand them completely. Right now I’m building hands-on skills through labs like this one, and I’m always looking to learn from people with more experience. If you see something I could improve or want to collaborate, I’m all ears.

contact details:

 https://www.linkedin.com/in/samuel-donkor-9258aa223?utm_source=share_via&utm_content=profile&utm_medium=member_ios
	•	Email: samuelcyber02@gmail.com
  
License
Educational project for CompTIA Security+ training. Performed in an isolated lab environment against lab-provided systems only.

Last Updated: August 2026
Status: Completed ✅ | Still Learning 📚
