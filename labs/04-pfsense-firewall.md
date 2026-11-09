# Lab 04: Firewall Administration with pfSense

## Objective

Build a pfSense firewall for the lab network, assign WAN and LAN interfaces, configure safe administration, create aliases and firewall rules, review NAT behavior, enable useful logging, and test the rules from a client.

This lab matters for IT support because many access problems are policy problems. A service can be installed correctly and still fail because a rule blocks the port, DNS points to the wrong resolver, NAT behaves in an unexpected way, or the client follows a different path than the technician assumed.

By the end of the guide, the client should receive access according to a written rule plan. Allowed traffic should pass. Selected traffic should be blocked and visible in the firewall log. The administrator should be able to explain rule order and restore access after a controlled test.

## Concepts explained

### Stateful inspection, interface direction, and rule order

pfSense tracks permitted connections. When a client begins an allowed connection, reply traffic that belongs to that connection can return without a separate rule for every reply packet. The first packet of a new connection still needs a matching rule on the interface where it enters pfSense.

Rules are evaluated on the interface where traffic enters pfSense. Client traffic toward the internet enters on LAN. Internet traffic toward a lab service enters on WAN. This distinction prevents a LAN problem from being treated with random WAN rules.

Rules on an interface are evaluated from top to bottom, and the first matching rule normally wins. A broad allow rule placed above a specific block rule can make the block rule useless. Write the rule plan before creating the rules.

### Default deny and aliases

Traffic that does not match an allow rule is blocked by default. During diagnosis, review the rule list and the firewall log together because the log shows the traffic and the rule that acted on it.

An alias is a named group of hosts, networks, ports, or related values. Aliases make rules easier to read and maintain. A rule for an IT Practice Hosts alias is clearer than several separate address entries.

### NAT

Network Address Translation, usually called NAT, changes source or destination addressing as traffic passes through the firewall. Outbound NAT lets private lab addresses share the WAN address. A port forward changes an inbound destination to an internal server and port. NAT and firewall permission are related but separate controls.

### Logging

A rule can record matching traffic in the firewall log. The log connects time, source, destination, port, interface, and rule. Log learning, denial, administration, and security review rules, while avoiding so much routine logging that important entries become hard to find.

## What you need

* A pfSense installation image from the official project source.
* A virtual machine with at least two network adapters.
* One adapter connected to the outside network through NAT or a bridged connection.
* One adapter connected to the private VMware lab LAN.
* A written IP plan for WAN, LAN, the domain controller, the osTicket server, and the client.
* A browser on a lab client for the pfSense webConfigurator.
* Snapshots before rule changes and console access through VMware if web access is lost.

## Before you start

Label the VMware adapters before installation and record which one faces the outside network. Choose a lab LAN subnet that does not overlap the outside network. Keep the VMware console available during rule work so a mistake does not remove the only management path.

## Build steps

### Part A: Install pfSense and assign interfaces

1. Create the pfSense virtual machine.
2. Attach the first adapter to the outside network and the second adapter to the private lab LAN, then boot from the pfSense installation image and complete the installation.
3. Remove the installation image when instructed and reboot.
4. At the console, assign the outside adapter as WAN and the private lab adapter as LAN, and set the LAN IPv4 address and subnet prefix from the written IP plan.
5. Leave the LAN gateway blank unless the design requires one, because pfSense is the gateway for the lab LAN.
6. Decide whether pfSense or the Windows domain controller will provide DHCP on the lab LAN, and enable only the planned service.
7. From the client, browse to the LAN address and open the webConfigurator.

[Screenshot to add: pfSense console showing WAN and LAN interface addresses]

### Part B: Complete setup and secure administration

1. Sign in to the webConfigurator with the default administrator account.
2. Complete the setup wizard.
3. Set the host name to pfsense, set the domain according to the lab plan, set the primary DNS server, and set the America/Chicago time zone.
4. Change the administrator password and store it in a password manager.
5. Confirm that the WAN type matches the outside connection.
6. Review the option that blocks private networks on WAN and keep it enabled when the design does not require private address access from that side.
7. Finish the wizard and reload when prompted.
8. Open System and then Advanced, review the webConfigurator protocol and port, and keep HTTPS for administration if it is configured.
9. Create a named administrator practice account if you want to separate practice from the built in admin account.
10. Do not expose the webConfigurator to the internet for this lab.

[Screenshot to add: pfSense dashboard after the setup wizard]

### Part C: Create aliases

1. Open Firewall and then Aliases.
2. On the IP tab, create Lab_Servers with the domain controller and osTicket server addresses, and create IT_Practice_Hosts with the lab client address used for administration practice.
3. On the Ports tab, create Web_Ports and add ports 80 and 443.
4. Save each alias and apply the changes.
5. Use names that state the purpose rather than names such as Test1.

### Part D: Write the LAN rule plan

| Order | Action | Source | Destination | Port | Purpose |
| ---: | --- | --- | --- | --- | --- |
| 1 | Pass with logging | IT_Practice_Hosts | pfSense LAN address | 443 | Protect and observe administration |
| 2 | Pass | Lab LAN network | Domain controller | 53 | Allow DNS to the lab resolver |
| 3 | Pass | Lab LAN network | Any | Web_Ports | Allow normal web access |
| 4 | Pass | Lab LAN network | Any | 123 | Allow network time traffic if external time servers are used |
| 5 | Block with logging | Lab LAN network | Any | 23 | Block Telnet for the security exercise |
| 6 | Block with logging | Any | Any | Any | Make the final denial visible during learning |

This is a planning model, not a set of rules to copy without review. DNS needs separate UDP and TCP rules. A final block rule makes every unlisted service fail, which is useful for learning but can add work. A first session can keep the default LAN allow rule and add the final block during a later session.

### Part E: Create and log LAN rules

1. Open Firewall and then Rules and select the LAN tab.
2. Review the anti lockout rule and the existing default rules.
3. Add the administration rule with Action set to Pass, Interface set to LAN, Address Family set to IPv4, and Protocol set to TCP.
4. Set Source to IT_Practice_Hosts, Destination to the pfSense LAN address or This Firewall as offered by the version, and Destination Port to HTTPS or 443.
5. Write a description that states the purpose and enable Log in the rule options.
6. Create a DNS rule with Protocol set to UDP and Destination Port set to DNS or 53, and create a second DNS rule for TCP if the design requires it.
7. Create the web rule with Protocol set to TCP and Destination Port set to Web_Ports.
8. Create the Telnet block rule with Action set to Block, Protocol set to TCP, Destination Port set to 23, and Log enabled.
9. Arrange the rules in the planned order and select Apply Changes.
10. Open Status and then System Logs and then Firewall to confirm that the rules loaded without errors.

[Screenshot to add: pfSense LAN rule list in planned order]

### Part F: Review NAT

1. Open Firewall and then NAT and select the Outbound tab.
2. Confirm whether the mode is Automatic outbound NAT rule generation, and leave automatic mode in place for the basic lab unless the exercise is specifically about manual NAT.
3. Review the generated mapping and confirm that the source is the lab LAN network and the translation address is the WAN address.
4. Do not create a port forward during the basic build.
5. For a later port forward exercise, write the security reason first, limit the source when the design allows it, review the associated firewall rule, test from the correct side of the firewall, and remove the practice rule when the exercise ends.

Outbound NAT should usually be invisible to the client. The client uses pfSense as its gateway, and pfSense translates the source address as traffic leaves through WAN.

### Part G: Test rules from the client

1. Run `ipconfig /all` and confirm the address, subnet mask, gateway, and DNS server match the plan.
2. Ping the pfSense LAN address.
3. Use `nslookup` to resolve the lab domain through the domain controller, open a web page through the firewall, and open the osTicket portal on the lab LAN.
4. Run `Test-NetConnection` to a selected destination on port 443, and run it again to a selected destination on port 23 for the block exercise.
5. Open the pfSense firewall log and find the block entry.
6. Record the time, source, destination, port, and rule description in the session log.
7. Change one rule only if the evidence shows that the plan is wrong, apply the change, and repeat the same test.

[Screenshot to add: pfSense firewall log showing the blocked Telnet test]

## Verification checks

| Checkpoint | Action | Expected evidence |
| --- | --- | --- |
| Interfaces | Open the pfSense dashboard | WAN and LAN show the planned addresses and link state |
| Gateway | Ping pfSense from the client | The LAN address replies |
| DNS | Run `nslookup` from the client | The domain controller returns the expected answer |
| Web access | Open an allowed web destination | The page loads through the firewall |
| Local service | Open the osTicket portal | The service loads from the lab LAN |
| Block rule | Test TCP port 23 | The connection fails and a matching log entry appears |
| Administration | Sign in from the practice host | The webConfigurator opens through the planned method |
| Recovery | Correct a rule or restore a snapshot in a drill | The client regains the planned access |

## Troubleshooting

**The client cannot reach pfSense.** Check the VMware adapter connection and the client IP settings. Confirm that the client is on the lab LAN, the pfSense LAN address and subnet match the plan, and the correct virtual adapter was assigned to LAN at the console.

**The client reaches pfSense but cannot reach the internet.** Check the WAN address and gateway on the dashboard. Confirm that the client uses pfSense as its default gateway and review the outbound NAT mode. Test by IP address before testing by name so routing and DNS remain separate questions.

**Web pages fail by name but work by address.** Treat the problem as DNS until evidence shows otherwise. Check the client DNS server, use `nslookup`, review the DNS rules, and confirm that the domain controller DNS service is operating.

**A rule change has no effect.** Select Apply Changes after saving. Confirm that the rule is on the interface where traffic enters pfSense and that its order is correct. Consider whether an existing state permits traffic that began before a new block rule.

**The block rule did not log.** Confirm that Log is enabled. Confirm that the test used the expected protocol and port and that an earlier allow rule did not match first. Review the log display filters.

**Web administration is lost after a rule change.** Use the VMware console. Review the LAN rules and anti lockout state or restore the snapshot taken before the change. Record the faulty rule before correcting it.

**A port forward does not reach the lab server.** Confirm that the server is listening on the target port and uses the expected gateway for return traffic. Review the NAT rule, associated firewall rule, destination address, and firewall log. Test from the correct network side because an inside test can follow a different path.

## Security+ tie in

This lab mainly supports Domain 3, Security Architecture, through secure network design, segmentation, firewall rules, NAT, management access, and logging.

It also supports Domain 4, Security Operations, because firewall logs become evidence during incident review and access troubleshooting. It supports Domain 2, Threats, Vulnerabilities, and Mitigations, when insecure services such as Telnet are blocked and broad or stale rules are reviewed.

Useful exam language includes least privilege, network segmentation, access control list, stateful inspection, default deny, management plane protection, logging, and rule review.

## Portfolio evidence checklist

* [Screenshot to add: pfSense dashboard with interface status]
* [Screenshot to add: Alias list]
* [Screenshot to add: LAN rule list]
* [Screenshot to add: Outbound NAT mode]
* [Screenshot to add: Firewall log entry for the blocked Telnet test]

## My session log

| Date | What I did | What broke | How I fixed it | What I learned |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
