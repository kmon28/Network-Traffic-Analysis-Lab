# Network Traffic Analysis & Active Directory Investigation

> **Hands-on network security portfolio:** Wireshark + Windows Server 2022 + Active Directory + VirtualBox, with packet-level troubleshooting, authentication-protocol analysis, capture engineering, and evidence-based incident investigation.

![Windows Server 2022](https://img.shields.io/badge/Windows_Server-2022-2874A6?style=flat-square)
![Wireshark](https://img.shields.io/badge/Wireshark-Packet_Analysis-1679A7?style=flat-square)
![Active Directory](https://img.shields.io/badge/Active_Directory-DNS_%26_IAM-4B5563?style=flat-square)
![VirtualBox](https://img.shields.io/badge/VirtualBox-Network_Lab-2874A6?style=flat-square)
![Evidence](https://img.shields.io/badge/Evidence-10_Screenshots_%2B_PCAP-218838?style=flat-square)
![Status](https://img.shields.io/badge/Project-Documented-218838?style=flat-square)

**Quick navigation:** [Results](#investigation-results-at-a-glance) · [Lab architecture](#lab-environment) · [Investigations](#investigation-workflow) · [Evidence](#evidence-index) · [Limitations](#evidence-and-limitations)

## Executive Summary
Analyzed network traffic in an isolated Windows Active Directory lab to understand normal domain communications, troubleshoot a failed ICMP test, and investigate unsuccessful TCP connections. The key outcomes were **a validated, subnet-scoped firewall remediation**, **a better packet-capture method for VM-to-VM traffic**, and **evidence-based interpretation of TCP/9997 retransmissions without labeling them malicious**.

> **Evidence standard:** The observations below are drawn from the lab notes and screenshots. The supplied PCAP is identified as ICMP evidence; packet numbers for other protocols refer to the original Wireshark investigation and are not independently revalidated from that PCAP. The Splunk explanation is a hypothesis, not process attribution.

## Investigation results at a glance

| Investigation | Observation | Outcome / interpretation |
|---|---|---|
| ICMP firewall | Four inbound Echo Requests with no replies before remediation | Scoped Windows Firewall rule; four successful replies on retest |
| VM capture visibility | Host-only interface missed VM-to-VM unicast traffic | VirtualBox NIC tracing provided an effective alternate observation point |
| DNS / AD discovery | A/AAAA lookups and domain-related SRV queries | Baseline for DNS resolution and domain controller discovery |
| TCP / Kerberos / SMB / LDAP | Handshake, TGS exchange, SYSVOL access, and successful LDAP search | Documented expected domain communication patterns |
| TCP/9997 | SYN retransmissions without an observed completed handshake | Connectivity issue investigated; process and root cause not conclusively attributed |

## Network architecture

```text
                     Isolated VirtualBox lab
               Host-only network: 192.168.56.0/24

  CLIENT01 (Windows 10)                 DC02 (Windows Server 2022)
       192.168.56.102   <---------->       192.168.56.101
       Domain joined                    AD DS / DNS / lab.local
              |                                 |
              +--------- VirtualBox ------------+
                           |
                    Dell host / Wireshark
                       192.168.56.1

  Host-only interface capture: incomplete VM-to-VM visibility
  CLIENT01 NIC trace: alternate capture point for VM traffic
  NAT connectivity: separately available when needed
```

## Lab environment

| System | Function | Host-only IP |
|---|---|---|
| DC02 | Windows Server 2022, AD DS, DNS | `192.168.56.101` |
| CLIENT01 | Domain-joined Windows 10 | `192.168.56.102` |
| Physical Dell | VirtualBox host, Wireshark | `192.168.56.1` |
| Domain | Active Directory | `lab.local` |

Host-only subnet: `192.168.56.0/24`. NAT connectivity was separately available when needed. The host-only adapter was named **Ethernet 2** on the Dell.

## Investigation workflow

### 1. Capture placement: a visibility limitation
Capturing on the Dell's VirtualBox host-only adapter did **not reliably show CLIENT01 ↔ DC02 unicast traffic**. Changing CLIENT01 Adapter 1 promiscuous mode from **Deny** to **Allow VMs** did not fix host capture visibility.

**Resolution:** Shut down CLIENT01, then enable VirtualBox NIC tracing on the host:

```cmd
cd "C:\Program Files\Oracle\VirtualBox"
VBoxManage modifyvm "CLIENT01" --nictrace1 on --nictracefile1 "C:\Users\Public\CLIENT01-DNS.pcap"
```

After restarting CLIENT01 and generating traffic, the NIC trace captured VM communications for analysis. **Lesson:** a packet absent at one observation point may still have traversed another segment. Capture placement affects conclusions.

### 2. ICMP failure → scoped firewall fix
**Evidence:** [ICMP validation screenshot](screenshots/01_icmp_request_reply_validation.png)

From DC02, `ping 192.168.56.1` initially produced **4 timeouts**. Wireshark on the Dell observed four inbound **ICMP Echo Requests (type 8)** from `192.168.56.101` without corresponding Echo Replies.

The Dell's Windows Defender Firewall was inspected. A **custom inbound ICMPv4 allow rule**, restricted to remote subnet `192.168.56.0/24`, was created for all profiles; existing File and Printer Sharing Echo Request rules remained disabled.

After retesting, **4 requests received 4 replies**; the trace showed **Echo Replies (type 0)**. This supports the destination firewall configuration as the cause of the observed failure, with a narrowly scoped fix instead of disabling the firewall.

### 3. DNS: ordinary resolution and AD discovery
**Evidence:** [DNS lookup](screenshots/02_Wireshark_DNS_Query_Response_Client01_DC02.png) · [AD discovery](screenshots/03_Wireshark_AD_DNS_Service_Discovery.png) · [additional AD DNS view](screenshots/03_Wireshark_AD_LDAP_DNS_Service_Discovery.png)

On CLIENT01, `nslookup google.com 192.168.56.101` succeeded. The filter `dns && dns.qry.name == "google.com"` isolated **A and AAAA requests and responses** between CLIENT01 and DC02. The resolver label `Unknown` in nslookup did not indicate lookup failure. Returned public IPs can change.

Filtering `dns.qry.name contains "lab.local"` showed **Active Directory service-discovery DNS traffic**, including LDAP/DC-related SRV names under `_tcp` and `_msdcs.lab.local`. This illustrates how a domain-joined workstation discovers directory services.

### 4. TCP: successful connection establishment
**Evidence:** [TCP handshake](screenshots/04_Wireshark_TCP_Three_Way_Handshake.png)

Packets **367–369** show a complete handshake:
- **367:** CLIENT01 `192.168.56.102:49692` → DC02 `192.168.56.101:135`, **SYN**
- **368:** DC02 → CLIENT01, **SYN/ACK**
- **369:** CLIENT01 → DC02, **ACK**

This baseline helps distinguish established connections from repeated connection attempts and resets.

### 5. Kerberos: service-ticket exchange
**Evidence:** [Kerberos TGS exchange](screenshots/05_Wireshark_Kerberos_TCG_Auhentication.png)

Filter: `kerberos`. Packet **415** was a **TGS-REQ** from CLIENT01 to DC02 on port **88**, realm `LAB.LOCAL`; packet **418** was the associated **TGS-REP**. A machine-account name appeared in the request. Kerberos provides ticket-based service authentication. Some `KRB5KDC_ERR_S_PRINCIPAL_UNKNOWN` errors were present, but **no malicious cause was established**.

### 6. SMB2: SYSVOL and domain policy
**Evidence:** [SMB/SYSVOL](screenshots/06_Wireshark_SMB_SYSVOL_Domain_Policy_Access.png)

Filter: `smb2`. Packet **469** was a **Session Setup Response** with `STATUS_SUCCESS`; packet **471** showed a successful **Tree Connect response** associated with `\\WIN-QDGUB4SSHAG.lab.local\SysVol`. Subsequent traffic referenced `lab.local\Policies`. This demonstrates client access to domain policy resources via SMB.

### 7. LDAP: successful directory search
**Evidence:** [LDAP search](screenshots/Wireshark_LDAP_AD_Search_Response_Success.png)

Filter: `ldap`. Packet **491** contained a root/base-object LDAP search request; packet **493** was the corresponding response. Wireshark showed **four results**, `searchResDone: success`, and approximately **805 μs** response time. LDAP provides directory queries; it is distinct from Kerberos ticket-based authentication.

### 8. TCP/9997: investigate, don't assume compromise
**Evidence:** [Retransmission analysis](screenshots/08_Wireshark_TCP_Retrasmission_Investigation.png)

Filter: `tcp.analysis.flags`. CLIENT01 repeatedly sent **SYN retransmissions** to DC02 **TCP/9997**. The investigated attempt had **no observed completed three-way handshake**, and **Follow TCP Stream** showed no application payload.

**Hypothesis:** TCP/9997 had been configured for Splunk Universal Forwarder → Splunk Enterprise traffic in this lab, so unsuccessful or stale forwarding connectivity was plausible. **Not proven:** the originating process was not correlated to an executable or PID; neither an attack nor a specific service root cause was established.

### 9. Capture-wide protocol overview
**Evidence:** [Protocol hierarchy](screenshots/09_Wireshark_Protocol_Hierarchy_Traffic_Overview.png)

**Statistics → Protocol Hierarchy** summarized **1,770 packets** after clearing the temporary `tcp.stream eq 36` display filter. Always verify whether statistics apply to the full capture or only displayed packets.

## Wireshark filter reference

| Filter | Investigation |
|---|---|
| `dns && dns.qry.name == "google.com"` | A/AAAA DNS lookup |
| `dns.qry.name contains "lab.local"` | AD service discovery |
| `tcp` | Handshake inspection |
| `kerberos` | Service-ticket exchange |
| `smb2` | SYSVOL and policy access |
| `ldap` | Directory search |
| `tcp.analysis.flags` | TCP anomalies |

## Key lessons
1. **Capture-point visibility:** VM-to-VM traffic was not reliably observable on the host-only interface; NIC tracing solved the collection problem.
2. **Packet-driven remediation:** observing requests without replies guided an ICMP firewall investigation and a subnet-scoped rule.
3. **Normal protocol baseline:** DNS, TCP, Kerberos, LDAP and SMB are essential to interpreting Windows domain activity.
4. **Avoid false positives:** TCP retransmissions and Kerberos errors warrant context and corroboration, not automatic attack labels.
5. **Evidence boundaries:** screenshots document protocol observations; an original PCAP permits independent packet-level verification.

## Evidence index

The following files are committed to this repository. Screenshots support the documented observations; the available PCAP specifically supports the ICMP investigation.

| Evidence | File |
|---|---|
| ICMP firewall validation | [Screenshot](screenshots/01_icmp_request_reply_validation.png) |
| DNS A/AAAA query and response | [Screenshot](screenshots/02_Wireshark_DNS_Query_Response_Client01_DC02.png) |
| AD DNS service discovery | [Screenshot](screenshots/03_Wireshark_AD_DNS_Service_Discovery.png) |
| Additional AD LDAP/DNS discovery | [Screenshot](screenshots/03_Wireshark_AD_LDAP_DNS_Service_Discovery.png) |
| TCP three-way handshake | [Screenshot](screenshots/04_Wireshark_TCP_Three_Way_Handshake.png) |
| Kerberos TGS exchange | [Screenshot](screenshots/05_Wireshark_Kerberos_TCG_Auhentication.png) |
| SMB2 SYSVOL policy access | [Screenshot](screenshots/06_Wireshark_SMB_SYSVOL_Domain_Policy_Access.png) |
| LDAP search response | [Screenshot](screenshots/Wireshark_LDAP_AD_Search_Response_Success.png) |
| TCP retransmissions | [Screenshot](screenshots/08_Wireshark_TCP_Retrasmission_Investigation.png) |
| Protocol hierarchy | [Screenshot](screenshots/09_Wireshark_Protocol_Hierarchy_Traffic_Overview.png) |
| ICMP capture | [Download PCAPNG](pcaps/ICMP_Firewall_Investigation.pcapng) |

## Reproduction and analyst validation

To reproduce the *method* in a controlled lab, deploy a Windows domain controller and domain-joined client on an isolated VirtualBox host-only subnet, generate DNS and domain logon traffic, and compare host-side Wireshark visibility with VM NIC tracing. Use a **test-only, subnet-scoped** ICMP firewall rule when validating the ping scenario; avoid disabling the host firewall wholesale.

For independent review of the included ICMP capture:

1. Open [the PCAPNG](pcaps/ICMP_Firewall_Investigation.pcapng) in Wireshark.
2. Apply `icmp` and compare Echo Request (`icmp.type == 8`) versus Echo Reply (`icmp.type == 0`) packets, source/destination IPs, and timing.
3. Correlate packet observations with the [ICMP screenshot](screenshots/01_icmp_request_reply_validation.png) and the written remediation timeline.
4. Do not infer that the ICMP PCAP contains Kerberos, LDAP, SMB, or TCP/9997 evidence; those observations are documented through separate screenshots and notes.

**Suggested follow-up validation for TCP/9997:** check `netstat -ano` or `Get-NetTCPConnection` on CLIENT01, map any owning PID to a process, inspect Splunk Universal Forwarder output configuration and service state, verify a listener on DC02, and repeat a timestamped capture. These are **next investigative steps, not actions claimed completed**.

## Evidence and limitations

**Original capture:** [ICMP_Firewall_Investigation.pcapng](pcaps/ICMP_Firewall_Investigation.pcapng). The screenshot evidence links above point to the images in `screenshots/`.

The `screenshots/` folder contains the ten Wireshark screenshots. The [ICMP firewall investigation PCAP](pcaps/ICMP_Firewall_Investigation.pcapng) is stored in `pcaps/`. The original broader VirtualBox NIC trace is **not asserted to be separately available here**. Packet numbers and detailed field values outside the ICMP investigation are retained from the original exercise notes and screenshots.

**Interview summary:** I built an Active Directory lab, analyzed authentication and directory protocols in Wireshark, diagnosed and fixed an ICMP firewall issue with packet evidence, changed my capture method when host-side visibility was inadequate, and investigated SYN retransmissions without overstating attribution.
