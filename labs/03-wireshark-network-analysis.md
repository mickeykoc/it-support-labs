# Lab 03: Network Analysis with Wireshark

## Objective

Use Wireshark on lab systems to capture normal network traffic, identify common protocols, apply capture and display filters, and build an evidence based diagnosis for basic connectivity problems.

A help desk technician does not need to read every packet in a large capture. The technician does need to know when packet evidence can answer a question, how to narrow a capture to the relevant conversation, and how to separate a missing request, a missing reply, a name resolution failure, a transport problem, and an application problem.

By the end of the guide, the lab should produce small practice captures for address assignment, DNS, web traffic, ICMP, and a failed connection attempt. Each capture should have notes that state the question, filter, evidence, and conclusion.

## Concepts explained

### Packet and protocol layers

A packet is a unit of data moving across a network. Wireshark separates it into protocol layers so the reader can inspect addressing, ports, flags, sequence information, and application data where the protocol exposes it.

### Capture filter

A capture filter limits which packets are saved during capture. It reduces file size on a busy network. Capture filters use syntax such as `host 192.0.2.10` or `port 53`.

### Display filter

A display filter changes which saved packets are shown after capture. It does not delete the hidden packets. Display filters use protocol and field names, such as `dns`, `tcp.port == 443`, or `ip.addr == 192.0.2.10`.

### Conversation

A conversation is the exchange between endpoints. A request with no reply points toward routing, firewall, service, or address problems. A completed connection followed by application errors points toward a different layer.

### TCP handshake

TCP normally begins with SYN, SYN and ACK, and ACK. Repeated SYN packets with no reply can mean the destination is unreachable, filtered, offline, or reached through the wrong path. A reset means a host or device answered and refused the connection.

### Encryption boundary

Wireshark can show encrypted traffic endpoints, timing, volume, handshake events, and resets. It should not be expected to reveal the content of correctly encrypted traffic. HTTPS analysis focuses on the connection rather than the page content.

### DNS evidence

A DNS query with no response differs from a response that says a name does not exist. An unexpected answer address raises another diagnosis path. Packet details help separate client settings, resolver state, record problems, and network path problems.

## What you need

* Wireshark installed on the Windows client or another lab machine.
* Npcap on Windows if the installer requests a capture driver.
* VMware network access to the lab LAN.
* A domain controller or another known lab system to test.
* The osTicket server or another local web service.
* A folder for captures with clear file names.
* Permission to capture only on a network you own or are authorized to test.

## Before you start

Use small captures. Start the capture, perform one planned action, stop the capture, and save it with a name that states the test. Keep notes beside the capture or record the conclusion in the session log.

Do not leave a general capture running while entering passwords or opening personal accounts. A capture can contain names, addresses, device details, and content from poorly secured services.

## Build steps

### Part A: Install and orient Wireshark

1. Install Wireshark on the lab client and accept Npcap if capture support is required.
2. Open Wireshark and identify interfaces with active traffic graphs.
3. Select the interface connected to the VMware lab network.
4. Open Capture and then Options and review the input, output, and capture filter fields.
5. Start a short capture and open a lab web page.
6. Stop the capture and open Statistics and then Protocol Hierarchy.

[Screenshot to add: Wireshark interface list with the lab adapter selected]

### Part B: Capture address assignment traffic

1. Start a capture using the capture filter `port 67 or port 68`.
2. On a disposable lab client, run `ipconfig /release` only if losing the current address during the exercise is acceptable.
3. Run `ipconfig /renew`.
4. Stop the capture after the client receives an address.
5. Apply the display filter `bootp`.
6. Find the Discover, Offer, Request, and Acknowledgement messages.
7. Identify the offered address, DHCP server, lease time, subnet mask, gateway, and DNS server in the packet details.
8. Run `ipconfig /all` and compare the client configuration with the packet evidence.

Wireshark uses the display filter name `bootp` for DHCP traffic because DHCP grew from BOOTP. The packet list may label the protocol as DHCP.

[Screenshot to add: Wireshark showing a DHCP exchange with the display filter applied]

### Part C: Capture DNS traffic

1. Clear the Windows DNS cache with `ipconfig /flushdns`.
2. Start a new capture.
3. Run `nslookup` for the lab domain or the osTicket server name.
4. Open the osTicket portal by name in a browser.
5. Stop the capture and apply the display filter `dns`.
6. Find the query and response pair.
7. Record the query name, query type, resolver address, response code, and answer.
8. Apply `dns.flags.response == 0` to show queries only.
9. Apply `dns.flags.response == 1` to show responses only.
10. Apply `dns.qry.name contains "osticket"`, replacing the text with the lab server name when needed.

If a browser used cached data or DNS over HTTPS, the normal DNS filter may not show the lookup. Use `nslookup` for a controlled test because it follows the system resolver path directly.

### Part D: Capture web traffic

1. Start a capture and open the osTicket portal or another lab web service.
2. Use a lab practice account only if the exercise requires sign in.
3. Stop the capture after the page loads.
4. Apply `tcp.port == 80` for plain HTTP or `tcp.port == 443` for HTTPS.
5. For HTTP, apply `http.request` to show requests.
6. Select one HTTP request and use Follow and then TCP Stream to inspect the exchange.
7. For HTTPS, identify the TCP handshake and the TLS Client Hello and Server Hello.
8. Record the destination address, port, TLS server name indication if present, and whether the session completed.

| Question | Display filter |
| --- | --- |
| Show one host only | `ip.addr == 192.0.2.20` |
| Show DNS only | `dns` |
| Show HTTPS port traffic | `tcp.port == 443` |
| Show connection starts | `tcp.flags.syn == 1 and tcp.flags.ack == 0` |
| Show resets | `tcp.flags.reset == 1` |
| Show ICMP traffic | `icmp` |
| Show Remote Desktop port traffic | `tcp.port == 3389 or udp.port == 3389` |

Replace example addresses and ports with the values used in the lab.

### Part E: Capture a failed connection

1. Choose a lab server and a TCP port that should be closed, or use the pfSense lab to block a selected service.
2. Record the intended closed service or firewall rule before testing.
3. Start a capture.
4. From the client, run `Test-NetConnection` with the computer name and port.
5. Stop the capture and filter for the destination address and port.
6. Determine whether the client sent repeated SYN packets, received a reset, received an ICMP message, or received no relevant reply.
7. Compare the packet evidence with the firewall log or service state.
8. Write the conclusion as a diagnosis supported by evidence.

Example conclusion language: The client sent TCP SYN packets to the server address and port. No SYN and ACK reply appeared. The firewall log recorded a block for the same source, destination, and port. The evidence supports a firewall rule as the cause.

[Screenshot to add: Wireshark showing repeated SYN packets with no reply]

## Verification checks

| Checkpoint | Action | Expected evidence |
| --- | --- | --- |
| Capture path | Start a capture and generate lab traffic | Packets increase on the selected interface |
| DHCP | Filter for `bootp` after an address renewal | The four stage DHCP exchange is visible |
| DNS | Filter for `dns` after `nslookup` | A query and response pair is visible |
| TCP | Follow a completed web connection | SYN, SYN and ACK, and ACK appear at the start |
| Display filters | Apply and clear several filters | Shown packets change while the saved capture remains intact |
| Failed connection | Compare the capture with service or firewall state | The conclusion matches the packet pattern |
| Notes | Review the session log | Each capture has a question, filter, evidence, and conclusion |

## Troubleshooting

### No packets appear

Confirm that the selected adapter is connected to the lab network. Generate known traffic with `ping` to the gateway or server. Confirm that the virtual machine network is connected in VMware. Check whether a capture filter excludes the expected traffic.

### The capture has too many packets

Stop the capture and begin with a display filter. For the next capture, use a narrow capture filter based on host or port and reduce the test to one DNS lookup or one page load.

### DNS traffic is missing

Clear the DNS cache and use `nslookup`. Check whether the browser uses DNS over HTTPS and whether the query used a different interface. Review both UDP and TCP port 53 because DNS can use either transport.

### HTTPS content cannot be read

That is expected behavior. Review endpoints, ports, TLS handshake messages, timing, resets, and total data. Use server logs or browser tools for application content questions.

### Requests appear without replies

Record the destination and time. Check destination power, service state, address, subnet mask, gateway, and firewall rules on the path. Compare the capture with the pfSense or host firewall log.

### A reset appears immediately

A reset usually means the destination or an intermediate device refused the connection. Confirm that the service is listening on the expected port and that the tested address is the intended address.

## Security+ tie in

This lab supports Domain 2, Threats, Vulnerabilities, and Mitigations, because packet evidence helps identify scanning, unexpected protocols, failed connections, and suspicious patterns.

It strongly supports Domain 4, Security Operations, through monitoring, log correlation, incident evidence, and network traffic analysis. It also supports Domain 3, Security Architecture, because DNS, DHCP, TCP, TLS, and address design become visible in operation.

Useful exam language includes packet capture, protocol analyzer, indicators of compromise, network monitor, escalation evidence, and encrypted traffic.

## Portfolio evidence checklist

* [Screenshot to add: Wireshark Protocol Hierarchy for a lab capture]
* [Screenshot to add: DHCP exchange filtered with bootp]
* [Screenshot to add: DNS query and response pair]
* [Screenshot to add: TCP handshake for a lab web connection]
* [Screenshot to add: Failed connection capture with repeated SYN packets]

## My session log

| Date | What I did | What broke | How I fixed it | What I learned |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
