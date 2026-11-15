# Network analyst: packet triage quick reference

Use this guide to review the saved investigation while separating direct observations from hypotheses.

| Signal | Wireshark filter | Question | Limitation |
|---|---|---|---|
| ICMP | `icmp` | Did Echo Requests receive replies? | A missing reply does not identify the blocking device |
| DNS | `dns` | Which query and response occurred? | A lookup does not prove an application connection |
| TCP SYN | `tcp.flags.syn == 1` | Was the handshake completed? | Retransmission alone does not attribute root cause |
| Kerberos | `kerberos` | What ticket exchange or error occurred? | Errors need protocol context |
| SMB2 | `smb2` | What operation and status were observed? | Visibility does not alone prove authorized access |
| LDAP | `ldap` | Which directory operation was observed? | Directory reads may be normal |

## Evidence review
1. Read the [results](../README.md#investigation-results-at-a-glance).
2. Open the [ICMP PCAP](../pcaps/ICMP_Firewall_Investigation.pcapng) for ICMP observations.
3. Inspect the [screenshots](../screenshots/) for DNS, Kerberos, SMB and LDAP evidence; these protocols are not asserted to be in the ICMP PCAP.
4. Record the source, timestamp, observation, inference and unresolved questions.

Capture location matters: host-only capture visibility was limited, and VirtualBox NIC tracing was used as an alternate observation point.
