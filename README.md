# Network Traffic Analysis & Active Directory Investigation

**Wireshark | Windows Server 2022 | Active Directory | VirtualBox | Network Troubleshooting**

## Executive summary
Analyzed network traffic in an isolated Windows Active Directory lab to understand normal domain communications, troubleshoot a failed ICMP test, and investigate unsuccessful TCP connections. The key outcomes were **a validated, subnet-scoped firewall remediation**, **a better packet-capture method for VM-to-VM traffic**, and **evidence-based interpretation of TCP/9997 retransmissions without labeling them malicious**.

> **Evidence standard:** The observations below are drawn from the lab notes and screenshots. The supplied PCAP is identified as ICMP evidence; packet numbers for other protocols refer to the original Wireshark investigation and are not independently revalidated from that PCAP. The Splunk explanation is a hypothesis, not process attribution.

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

## Evidence and limitations

**Original capture:** [ICMP_Firewall_Investigation.pcapng](ICMP_Firewall_Investigation.pcapng). The screenshot evidence links above point to the original uploaded images in the repository root.

The repository root contains the ten supplied Wireshark screenshots. The [ICMP firewall investigation PCAP](ICMP_Firewall_Investigation.pcapng) is included in the repository root. The original broader VirtualBox NIC trace is **not asserted to be separately available here**. Packet numbers and detailed field values outside the ICMP investigation are retained from the original exercise notes and screenshots.

**Interview summary:** I built an Active Directory lab, analyzed authentication and directory protocols in Wireshark, diagnosed and fixed an ICMP firewall issue with packet evidence, changed my capture method when host-side visibility was inadequate, and investigated SYN retransmissions without overstating attribution.
