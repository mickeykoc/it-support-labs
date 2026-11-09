# IT Support Labs

## About me

I am Michael Nwokocha, an IT support specialist in training based in San Marcos, Texas.

Before moving toward IT support, I founded Visionary Vessels, where I onboarded 20+ clients to the GoHighLevel customer relationship management platform. That work included training, troubleshooting, account setup, workflow support, and patient explanations for people with very different levels of technical confidence.

I am now building the technical foundation for my first IT help desk role. This repository documents the home lab procedures I use to practice Windows administration, help desk workflow, network analysis, firewall administration, troubleshooting, and security operations.

## Purpose of this repository

These files are lab guides and study documentation. Each guide explains the purpose of a system, the concepts behind it, the build procedure, the checks that prove the system works, and the diagnosis path to follow when it does not.

The guides are written for repeat use. I can rebuild a lab from the steps, compare a broken system against the verification checks, and record each working session in the log at the end of the guide.

The session logs are intentionally blank until a lab is run. They are the place to record dates, actions, faults, fixes, and lessons from real practice.

## Lab guides

1. [Active Directory on Windows Server 2022](labs/01-active-directory.md)
2. [Help Desk Practice with osTicket](labs/02-helpdesk-osticket.md)
3. [Network Analysis with Wireshark](labs/03-wireshark-network-analysis.md)
4. [Firewall Administration with pfSense](labs/04-pfsense-firewall.md)

## Skills matrix

| Tool or platform | Skills practiced | Guide |
| --- | --- | --- |
| VMware | Virtual machines, snapshots, virtual networks, resource settings | All guides |
| Windows Server 2022 | Server roles, updates, static addressing, administration | Active Directory |
| Active Directory | Domains, organizational units, users, groups, account policy | Active Directory |
| Group Policy | Password policy, lockout policy, policy scope, policy refresh | Active Directory |
| PowerShell | Repeatable administration, CSV import, account queries | Active Directory |
| osTicket | Ticket intake, triage, assignment, service levels, escalation notes | Help Desk Practice |
| Wireshark | Packet capture, display filters, protocol analysis, evidence collection | Network Analysis |
| pfSense | Interface setup, firewall rules, NAT, aliases, logs, rule testing | Firewall Administration |
| DNS and DHCP | Name resolution, address assignment, scope checks, lease review | Multiple guides |
| Documentation | Build notes, screenshots, verification evidence, session logs | All guides |

## Lab topology

The lab uses one VMware host and a private virtual network.

```text
Home network or internet connection
                |
          pfSense WAN interface
          pfSense firewall VM
          pfSense LAN interface
                |
        VMware private LAN segment
        |              |              |
 Windows Server     Windows       osTicket
 2022 domain        client VM     server VM
 controller
```

The pfSense virtual machine has two network adapters. One adapter faces the outside network. The other adapter serves the private lab LAN.

The Windows Server 2022 virtual machine serves as the domain controller for the Active Directory lab. It can also provide DNS for lab systems.

The Windows client virtual machine is the normal user workstation. It is used for domain sign in tests, Group Policy tests, ticket scenarios, packet captures, and firewall rule tests.

The osTicket server virtual machine hosts the practice help desk. It receives the requests created during ticket scenarios.

All lab systems should use private lab addresses. Production credentials, customer information, and employer information do not belong in this environment.

## Recommended build order

1. Create the VMware networks and take clean snapshots of every virtual machine.
2. Build the pfSense firewall and confirm that the lab LAN can reach the firewall.
3. Install Windows Server 2022 and promote it to a domain controller.
4. Join the Windows client to the domain and test a standard user account.
5. Install osTicket and configure the help desk workflow.
6. Use Wireshark during the other labs to observe DNS, DHCP, authentication, web, and blocked traffic.

## Working method

Every lab follows the same support method.

1. State the goal in plain language.
2. Record the starting condition.
3. Change one major setting at a time.
4. Test the change from the point of view of a user.
5. Capture evidence with a screenshot or a command result.
6. Record faults in the session log before trying random fixes.
7. Write the final diagnosis in cause and effect language.

This method matters because help desk work is not only about knowing settings. It is also about proving what changed, limiting risk, communicating clearly, and leaving notes that another technician can follow.

## Documentation standard

Each guide includes placeholders for screenshots. A useful screenshot should show the setting or result named in the placeholder. It should not expose passwords, personal email content, customer data, recovery keys, or private network information from outside the lab.

Commands and settings should be copied carefully. If a value differs in my lab, the session log should say what value was used and why.

## Safety and scope

* Practice only on systems I own or systems I am authorized to administer.
* Take a snapshot before major role, policy, firewall, or update changes.
* Use test users for lockout and password exercises.
* Do not use a real person as the subject of a security test case.
* Do not capture traffic on a network unless I own it or have clear permission.
* Keep default gateway and DNS changes inside the lab network.
* Store lab passwords in a password manager rather than in this repository.

## Current focus

My current goal is a first IT help desk or technical support role.

I am studying for CompTIA Security+ SY0-701, with the exam planned for November 2026. The certification is in progress and is not listed as held.

The labs in this repository connect that study to daily technical practice. Active Directory supports identity and access management. osTicket supports incident workflow and communication. Wireshark supports network diagnosis. pfSense supports network security controls and log review.

## Repository layout

```text
README.md
labs/
  01-active-directory.md
  02-helpdesk-osticket.md
  03-wireshark-network-analysis.md
  04-pfsense-firewall.md
```
