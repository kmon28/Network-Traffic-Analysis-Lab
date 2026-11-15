# Network Traffic Analysis — Technical Interview Study Guide

This guide prepares the author to defend the **documented network investigation**, not claim additional lab work. Read the [investigation](../README.md) and [original ICMP capture](../pcaps/ICMP_Firewall_Investigation.pcapng) before interviews.

## 60-second project explanation

I analyzed network traffic in an isolated Windows Active Directory lab with Wireshark. I investigated ICMP requests that received no replies, validated a subnet-scoped firewall change, and improved capture visibility by switching from the host-only interface to VirtualBox NIC tracing. I examined DNS/AD discovery, TCP handshakes, Kerberos, SMB2/SYSVOL, LDAP, and repeated TCP/9997 SYN attempts. I treated the unconfirmed TCP/9997 process attribution as a hypothesis rather than claiming an intrusion.

## What a SOC or network security interviewer may ask

### Why was the first capture incomplete?
A host-only adapter on the physical host did not reliably expose unicast traffic exchanged between the VMs. Changing promiscuous mode did not resolve that visibility gap. VirtualBox NIC tracing on CLIENT01 provided a different observation point. A missing packet in one capture is not proof it never traversed the network.

### What proved the ICMP remediation worked?
The investigation observed Echo Requests without Echo Replies before the firewall change, then requests **and** replies after a custom inbound ICMPv4 rule scoped to the lab subnet. This supports the firewall configuration as the cause of the observed symptom. The [PCAP](../pcaps/ICMP_Firewall_Investigation.pcapng) is the original independently inspectable packet artifact; the [screenshot](../screenshots/01_icmp_request_reply_validation.png) records the before/after investigation.

### How do you distinguish a handshake from a failed connection?
A completed TCP three-way handshake contains SYN, SYN/ACK, then ACK. Repeated SYN retransmissions with no observed SYN/ACK show that the initiating host did not observe a response during that attempt; they do not establish whether a firewall, unavailable listener, routing issue, capture gap, or other cause is responsible.

### Why does DNS matter to Active Directory?
Clients query A/AAAA records for address resolution and SRV records to locate directory-related services and domain controllers. Successful DNS discovery is a prerequisite for many domain workflows but does not itself prove that a subsequent authentication or directory operation succeeded.

### What is the difference between Kerberos, LDAP, and SMB2?
Kerberos exchanges authentication tickets, LDAP queries directory information, and SMB2 provides file/share access such as SYSVOL. The recorded TGS exchange, LDAP search success, and SMB2 status are distinct observations and should not be conflated.

### Did TCP/9997 prove Splunk forwarding was responsible?
No. Splunk forwarding was a plausible configuration-based explanation, but the capture did not establish the owning PID or executable. Process-to-connection mapping, listener checks, service configuration, and a timestamped repeat capture would be needed. Those steps are **proposed follow-up work**, not completed evidence.

### Does this demonstrate detection of a production attack?
No. It demonstrates packet analysis, network troubleshooting, Windows/AD protocol interpretation, and cautious triage in a controlled lab. The documented traffic does not establish a production incident or malicious activity.

## Role-specific interview emphasis

| Reviewer | What to show in this repository | Honest boundary |
| --- | --- | --- |
| SOC analyst / team lead | Evidence-first triage; distinguishing anomalous traffic from confirmed compromise | No malicious actor or production incident confirmed |
| Network security analyst | Capture placement, ICMP firewall validation, handshake and retransmission reasoning | Only the ICMP PCAP is committed |
| Security engineer | Scoped firewall remediation, VM network topology, AD protocol flows | No claim of enterprise-scale deployment |
| Detection engineer | Understanding of protocol signals and false-positive avoidance | No packet-based IDS rule or detection test suite is provided |
| Hiring manager | Clear problem, action, verified outcome, and follow-up questions | A lab is not equivalent to paid SOC experience |

## Evidence and preparation checklist

- [ICMP PCAP](../pcaps/ICMP_Firewall_Investigation.pcapng): independently inspectable ICMP traffic
- [Screenshot index](../README.md#evidence-index): ten investigation images for the wider protocol work
- [Technical reviewer guide](reviewer-guide.md): five-minute evidence path
- [Packet triage reference](packet-triage-quick-reference.md): filters and interpretation limits

**Interview method:** observation → competing explanations → test or collection method → supported conclusion → uncertainty → next investigative step.
