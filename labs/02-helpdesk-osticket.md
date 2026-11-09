# Lab 02: Help Desk Practice with osTicket

## Objective

Install and configure osTicket as a practice help desk, then work realistic tickets from intake through resolution or escalation.

The main lesson is the support process around the system: asking useful questions, recording facts, choosing a priority, assigning ownership, working within a service level agreement, communicating with the requester, documenting the diagnosis, and writing an escalation note that another technician can use without starting over.

By the end of the guide, the help desk should have departments, agents, roles, help topics, SLA plans, and practice tickets for a password reset, an account lockout, a printer issue, and a VPN access request.

## Concepts explained

### Ticket and intake

A ticket is the durable record of a request or incident. It should identify the requester, problem, impact, steps already tried, owner, priority, work history, and final resolution.

Intake establishes who is affected, what changed, what the impact is, what has already been tried, and how the technician can reach the requester. Good intake prevents a vague request from becoming an unfocused repair attempt.

### Triage

Triage sets urgency, impact, priority, category, and assignment. Priority should reflect the work impact rather than the tone or volume of the requester message.

### Departments, agents, roles, teams, and help topics

A department receives tickets for a support area. An agent is a staff account that works tickets. A role controls what the agent can see and change. A team groups agents for assignment and escalation.

A help topic classifies a request and can route it to a department and SLA plan. Good topics also tell the requester what information to provide at the start.

### Service level agreement and escalation notes

A service level agreement, usually called an SLA, defines response and resolution targets. In osTicket, an SLA plan can set a grace period and control overdue ticket handling. An SLA is a commitment to manage time, ownership, and communication even when a repair depends on another team or a vendor.

An escalation note transfers knowledge. It states the issue, impact, confirmed facts, tests and results, attempted changes, current state, risks, and the exact help requested from the next level.

## What you need

* A separate virtual machine for the osTicket server.
* A supported web server, database, and PHP environment for the selected osTicket release.
* The current osTicket release from the official project source.
* A browser on the Windows client virtual machine.
* Practice requester and agent accounts.
* The Active Directory lab if account scenarios should match its test users.
* Snapshots before installation and after configuration.

## Installation overview

Package names vary by operating system and osTicket version, so follow the requirements shown by the downloaded release.

1. Create a virtual machine and attach it to the lab LAN.
2. Install the operating system and available updates.
3. Install a web server, such as Apache or Internet Information Services, and install a supported PHP version with the extensions required by the installer.
4. Install MySQL or MariaDB.
5. Create an empty database and a database user with permission limited to that database.
6. Copy the osTicket files into the web root or into a dedicated help desk folder, and set the file permissions required by the installer.
7. Open the osTicket setup address in a browser and complete the installer.
8. Record the staff control panel address, usually the web address followed by `/scp`, and remove or protect the setup folder if the installer instructs you to do so.
9. Sign in as the administrator, replace any temporary credential, and take a snapshot after the empty help desk opens.

[Screenshot to add: osTicket staff control panel dashboard after installation]

## Build steps

### Part A: Set the help desk identity

1. Sign in to the staff control panel.
2. Open Settings and then System.
3. Set the help desk name to IT Support Lab.
4. Set the default department and the lab time zone, confirm the date and time format, and save the settings.

### Part B: Create departments

| Department | Purpose |
| --- | --- |
| Technical Support | General incidents and how to questions |
| Account Access | Password, account state, group access, and sign in issues |
| Network Support | VPN, DNS, connectivity, and firewall requests |

Create each department, write a clear description, and decide whether it appears on the requester portal. In this small lab, all three can be visible so requesters can practice choosing the right path.

### Part C: Create roles and agents

1. Open Agents and then Roles.
2. Create a Level 1 Support role with permission to create, view, reply to, assign, transfer, and close tickets in assigned departments.
3. Do not give Level 1 permission to delete tickets or change system wide settings.
4. Create a Level 2 Support role with broader department access for escalated work.
5. Create one Level 1 practice agent and one Level 2 practice agent, and assign each agent to a primary department.
6. Add each agent to other departments only when a scenario requires it, create a Support Team, and add the Level 2 agent for escalation practice.
7. Sign in as each agent and confirm that the visible queues and controls match the role.

[Screenshot to add: osTicket agent list showing Level 1 and Level 2 practice agents]

### Part D: Create help topics

| Help topic | Department | Requester information to request |
| --- | --- | --- |
| Password Reset | Account Access | Service affected, user name, exact message, approved contact method |
| Account Lockout | Account Access | Workstation, start time, recent password change, other signed in devices |
| Printer Issue | Technical Support | Printer name, location, error message, test page result, effect on other users |
| VPN Access Request | Network Support | Business reason, approver, dates needed, device type |
| General Support | Technical Support | Goal, impact, error message, steps already tried |

Create each topic, add the description, and route it to the department shown. A good topic description reduces follow up questions and improves the first work note. The requester information in the table states what the topic description should ask for at intake.

### Part E: Configure SLA plans

Create these practice plans in Settings and then SLA. Set each grace period to match the resolution target and choose how overdue tickets should be handled.

| SLA plan | First response target | Resolution target | Use |
| --- | ---: | ---: | --- |
| Critical | 1 hour | 8 hours | Service down or many users blocked |
| Standard | 4 hours | 24 hours | One user blocked or a normal incident |
| Request | 8 hours | 72 hours | Planned access or service request |

Apply Critical only when impact and urgency justify it. Apply Request to an approved access workflow with a future start date. These are lab practice values. A real organization sets targets through service commitments, staffing, contracts, and support hours.

[Screenshot to add: osTicket SLA plan list]

### Part F: Practice the ticket lifecycle

1. Create the ticket as the requester or through the client portal, read it, and add an internal note that separates known facts from missing facts.
2. Ask the requester for missing information when needed.
3. Set the help topic, department, priority, SLA plan, and assignee.
4. Record each test and result while work happens, and post a requester response when a delay or next step needs explanation.
5. Resolve the issue or escalate it with a complete note.
6. Close with a summary that names the cause, the fix, and the requester confirmation when confirmation was part of the scenario.

## Practice scenarios

**Password reset.** The requester cannot sign in and believes the password is correct. Ask which service is affected, what user name is in use, what exact message appears, and which approved verification method is available. Verify identity under the lab rule, check the account state in Active Directory, reset the password if the check passes, require a change at next logon, and remain with the requester during the sign in test. Close only after the result is known.

**Account lockout.** The requester reports repeated lockouts after using a shared workstation. Ask when the problem began, whether the password changed recently, and whether old sessions or saved credentials may exist on another device. Check the account state and failed logon evidence, unlock the account for testing, remove one suspected cause at a time, and test from one workstation. Escalate if the lockout returns with no clear source or if the pattern raises a security concern.

**Printer issue.** Documents remain in the queue although the printer appears in Windows. Ask for the printer name, location, error message, test page result, and whether another user can print. Check printer power and network state, the Windows queue, and the print spooler service. Test from a second user or workstation when available. Record the model, address, port, driver, and queue state before removing the printer and adding it again. Escalate with those facts if the fault remains.

**VPN access request.** The requester needs VPN access for a project. Ask for the business task, approver, dates needed, and device type. Do not grant access from the ticket alone. Record the approval evidence, assign the ticket to Network Support, add the test user to the practice VPN group only after approval, and test the connection. Record the group change and set a review point if access is temporary.

## How to write escalation notes

Use this structure for every escalation.

```text
Issue summary:
Business impact:
Requester and affected systems:
Facts confirmed / Tests performed and results:
Changes attempted / Current state:
Security concerns / Exact help requested:
```

A strong lockout escalation would state that the account is enabled, the password reset completed, the requester signed in once from the known client, and a later attempt from the shared workstation caused another lockout. It would ask the next technician to review domain controller events and help identify the source workstation or saved session. Replace model statements with facts from the actual ticket before sending an escalation.

## Verification checks

| Checkpoint | Action | Expected evidence |
| --- | --- | --- |
| Installation | Open the client portal and staff panel | Both pages load without installer errors |
| Departments | Create one test ticket for each department | Each ticket appears in the selected department |
| Roles | Sign in as the Level 1 agent | System settings and delete controls are unavailable |
| Help topics | Submit each practice topic | Tickets route to the planned departments |
| SLA | Create a ticket with each plan | Due time reflects the selected plan |
| Assignment | Transfer a ticket to the Level 2 agent | Ticket history records the transfer |
| Closure | Resolve and close a practice ticket | Status and resolution summary are recorded |

## Troubleshooting

**The installer reports a missing PHP extension.** Read the exact extension name, install or enable it in the PHP configuration, restart the web server, and run the installer check again before changing database settings.

**osTicket cannot connect to the database.** Confirm the database name, user, password, host, and service state. Test permission on the osTicket database only. Do not solve the problem by giving the account control over every database.

**Agents cannot see a department queue.** Check the agent department access, the role permissions, and the queue filter. Confirm that the ticket belongs to the department the agent is viewing.

**Email settings fail during practice.** Record email as a separate configuration exercise. The core lab can still pass if the portal, staff panel, assignment, notes, SLA, and closure workflow operate correctly.

**The SLA due time looks wrong.** Check the help topic default, ticket SLA, department settings, and server time zone. Change one value at a time and create a new test ticket to verify the result.

## Security+ tie in

This lab mainly supports Domain 4, Security Operations, through incident workflow, evidence handling, escalation, communication, and lessons learned.

It also supports Domain 5, Security Program Management and Oversight, through service commitments, role design, approval records, and operational reporting. The account scenarios support Domain 1 and Domain 4 concepts for identity verification, least privilege, account recovery, and access approval.

## Portfolio evidence checklist

* [Screenshot to add: osTicket department list]
* [Screenshot to add: Help topic list]
* [Screenshot to add: SLA plan settings]
* [Screenshot to add: Practice ticket timeline with an internal note]
* [Screenshot to add: Completed escalation note]

## My session log

| Date | What I did | What broke | How I fixed it | What I learned |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
