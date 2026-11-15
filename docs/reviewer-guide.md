# Network Traffic Analysis — Technical Reviewer Guide

## Five-minute review path

1. Read the [investigation results](../README.md#investigation-results-at-a-glance) to identify the tested questions and conclusions.
2. Inspect [ICMP request/reply evidence](../screenshots/01_icmp_request_reply_validation.png) and the [original ICMP capture](../pcaps/ICMP_Firewall_Investigation.pcapng). The observed before/after result supports the subnet-scoped firewall fix.
3. Examine [DNS resolution](../screenshots/02_Wireshark_DNS_Query_Response_Client01_DC02.png), [AD service discovery](../screenshots/03_Wireshark_AD_DNS_Service_Discovery.png), [TCP handshake](../screenshots/04_Wireshark_TCP_Three_Way_Handshake.png), and [Kerberos exchange](../screenshots/05_Wireshark_Kerberos_TCG_Auhentication.png).
4. Read the [capture-placement limitation](../README.md#1-capture-placement-a-visibility-limitation) before interpreting missing packets.
5. Review the [TCP/9997 investigation](../README.md) and its unresolved attribution: retransmissions alone do not establish a process owner, malicious intent, or a root cause.

## Evidence provenance

| Artifact | What it can support | What it cannot independently support |
| --- | --- | --- |
| ICMP PCAP | ICMP packet-level verification of the firewall investigation | Kerberos, SMB, LDAP, or TCP/9997 packet claims |
| Wireshark screenshots | Visible protocol fields and packet observations captured during the lab | Reanalysis of complete packet streams not supplied in the repository |
| Written notes | Recorded investigation steps, hypotheses, and limitations | Independent proof of observations not preserved in original artifacts |

## Technical discussion prompts

- Why can a host-only adapter capture miss VM-to-VM unicast traffic?
- What packet evidence distinguishes a complete TCP three-way handshake from repeated SYN retransmissions?
- Why does an SRV DNS query matter for Active Directory service discovery?
- Why should a scoped firewall rule be preferred over disabling the firewall?
- What additional endpoint and listener telemetry would be needed to attribute TCP/9997 retransmissions?

**Scope:** This is a controlled lab investigation, not a production intrusion. Historical observations are credited as lab work; claims about the available repository artifacts are limited to what those artifacts actually show.
