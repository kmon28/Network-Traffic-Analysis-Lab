# Technical Study Guide — Security Lab Portfolio

This companion is for **personal review and interview preparation**. The public READMEs and incident reports prioritize investigation findings, evidence, and engineering reasoning.

## Five-minute review routine

1. Explain the lab's **objective and threat model** in one sentence.
2. Identify the **data source**: Sysmon, Windows Security, Task Scheduler, Linux journal, or PCAP.
3. Describe **what changed or triggered detection** and what evidence proves it.
4. Explain **why each command or filter was selected**.
5. Distinguish **observed behavior, interpretation, and unproven hypotheses**.
6. State the **response and verification**, or why containment was unnecessary.
7. Describe **one production improvement** without claiming it was already implemented.

## Common technical questions

<details><summary>Why do we need a baseline?</summary>

A baseline establishes expected state or behavior. A later observation can then be compared against it to identify changes. A change is a lead, not proof of an attack.

</details>

<details><summary>What makes a detection a true positive when the case is benign?</summary>

A detection can correctly identify its targeted behavior while that behavior is authorized. Detection accuracy and incident disposition are separate judgments.

</details>

<details><summary>How should screenshots be explained?</summary>

State the source, relevant fields, timing, result, and precise conclusion. Avoid claiming that a screenshot proves more than it shows.

</details>

<details><summary>What is the difference between detection, containment, and verification?</summary>

Detection flags activity. Containment limits or stops it. Verification checks that the intended control took effect. None automatically establishes a fully clean host.

</details>

<details><summary>What is a good answer when a tool or data source was unavailable?</summary>

Explain the limitation, use accessible alternative evidence where appropriate, and identify which conclusion could not be independently verified.

</details>

## Repository-specific review

- [Kali Linux investigations](https://github.com/kmon28/Kali-Linux-Security-Assessment-Lab): systemd persistence, SSH log parsing, Windows account discovery, scheduled tasks, SMB and Kerberos.
- [Windows SOC investigations](https://github.com/kmon28/Windows-SOC-Detection-Lab): Sysmon, Windows Security events, PowerShell detection, Active Directory and historical Splunk analytics.
- [Network traffic analysis](https://github.com/kmon28/Network-Traffic-Analysis-Lab): Wireshark capture placement, ICMP, DNS, TCP, Kerberos, SMB, LDAP and retransmissions.

**Study principle:** Be able to say **what I saw → why I investigated → what I tested → what the evidence proves → what remains uncertain**.
