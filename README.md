# Firewall-Configuration-and-Network-Defense
# Firewall Configuration & Intrusion Prevention Lab

## Objective

The Firewall Configuration & Intrusion Prevention Lab focused on implementing and analyzing host-based firewall controls to reduce a vulnerable system's exposed attack surface.

Using **Kali Linux, Metasploitable 2, Nmap, and iptables**, I first performed network reconnaissance to establish a baseline of exposed services. I then developed a least-privilege firewall policy that used default-deny filtering, stateful inspection, and explicit rules to restrict high-risk services while maintaining secure SSH administration.

The lab also explored **Windows Defender Firewall**, Intrusion Prevention System (IPS) concepts, and defense-in-depth security. This hands-on experience strengthened my understanding of firewall rule configuration, network access control, attack surface reduction, stateful packet filtering, and layered network defense.

---

## Skills Learned

- Configured Linux host-based firewall rules using `iptables`.
- Applied default-deny and least-privilege network access policies.
- Performed network reconnaissance and service enumeration using Nmap.
- Identified unnecessary and high-risk exposed network services.
- Implemented stateful firewall inspection for established connections.
- Restricted insecure services including FTP and Telnet.
- Maintained secure remote administration through SSH.
- Validated and reviewed firewall configurations.
- Compared firewall filtering with Intrusion Prevention System (IPS) capabilities.
- Strengthened understanding of defense-in-depth security.

---

## Tools Used

- **Kali Linux** — Used for network reconnaissance and security testing.
- **Metasploitable 2** — Intentionally vulnerable Linux system used as the firewall target.
- **Nmap** — Used for port scanning, service discovery, and version enumeration.
- **iptables** — Used to implement Linux host-based firewall rules and access-control policies.
- **SSH** — Used for encrypted remote administration of the target system.
- **Windows Defender Firewall** — Examined as a Windows host-based firewall solution.

---

## Steps

### 1. Established the Pre-Firewall Baseline

I began the assessment by performing a comprehensive **Nmap service and version scan** against the Metasploitable 2 system at `192.168.1.4`.

The scan identified numerous exposed services, including:

- FTP
- SSH
- Telnet
- SMTP
- DNS
- HTTP
- Samba/SMB
- NFS
- MySQL
- PostgreSQL
- VNC
- IRC
- Java RMI
- Legacy R-services
- Metasploitable bindshell

<img width="975" height="773" alt="image" src="https://github.com/user-attachments/assets/a016c076-2c1e-4e23-ae7e-e41cbfe6beb9" />
)

**Figure 1 — Initial Nmap service enumeration.** The baseline scan revealed numerous exposed network services and provided visibility into the target's attack surface before firewall restrictions were implemented.

The scan also identified service versions, which provided additional information about the software operating behind each exposed port.

<img width="975" height="774" alt="image" src="https://github.com/user-attachments/assets/ecbe89d9-5123-4481-b90f-cf7a9c4f8a07" />
)

**Figure 2 — Complete Nmap service enumeration.** Additional exposed services included database servers, remote-access services, RPC services, IRC, Apache Tomcat, and the Metasploitable root bindshell.

The full assessment identified **29 exposed TCP services**, demonstrating how unnecessary and legacy services can significantly increase a system's potential attack surface.

---

### 2. Assessed High-Risk Network Services

After establishing the baseline, I evaluated the exposed services to identify those presenting significant security concerns.

Three important findings were:

#### Bindshell — TCP 1524

The Metasploitable root shell represented a critical security concern because it exposed privileged system access and was unnecessary for normal system operation.

#### Telnet — TCP 23

Telnet provides remote administration without encrypting session traffic. This creates a confidentiality risk because credentials and commands can potentially be intercepted.

#### UnrealIRCd — TCP 6667/6697

The exposed IRC service represented another unnecessary service that increased the target's attack surface.

The assessment reinforced the **principle of least functionality**: systems should expose only the network services required for legitimate operations.

---

### 3. Established Secure Remote Administration

Before restricting incoming network access, I verified that the Metasploitable 2 system could be remotely administered through **SSH**.

```bash
ssh msfadmin@192.168.1.4
```

<img width="975" height="536" alt="image" src="https://github.com/user-attachments/assets/0778f73a-2922-4a50-8abc-d17106277dad" />
)

**Figure 3 — SSH connection to Metasploitable 2.** SSH provided an encrypted method for remote administration and was selected as the essential management service that would remain accessible through the firewall.

Maintaining SSH access was important because the firewall policy was designed to restrict unnecessary services without preventing legitimate administrative access.

---

### 4. Implemented a Default-Deny Firewall Policy

I then began configuring the Linux host-based firewall using `iptables`.

Existing firewall rules and user-defined chains were cleared:

```bash
sudo iptables -F
sudo iptables -X
```

A default-deny policy was then applied to incoming traffic:

```bash
sudo iptables -P INPUT DROP
```

<img width="975" height="549" alt="image" src="https://github.com/user-attachments/assets/c5257467-a0f6-406e-b70f-d7610c74229c" />
)

**Figure 4 — Beginning the iptables firewall configuration.** Existing rules were cleared and the INPUT chain was changed to a default `DROP` policy.

A **default-deny** approach improves security by reversing the traditional model of allowing traffic unless it is specifically blocked.

Instead, network traffic is denied unless a rule explicitly authorizes it.

This approach supports both **least privilege** and **least functionality** by limiting network access to approved services.

---

### 5. Implemented Stateful Inspection

The firewall was configured to permit traffic belonging to connections that were already established or related to legitimate connections.

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Stateful inspection allows the firewall to track connection information rather than treating every packet independently.

This allows legitimate response traffic to return to the system while unsolicited inbound connections remain restricted by the default-deny policy.

---

### 6. Allowed Essential SSH Traffic

Because SSH was selected as the secure remote-management protocol, TCP port 22 was explicitly permitted:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

This allowed authorized administrators to continue remotely managing the system while other incoming services remained restricted.

The configuration demonstrates an **allowlist approach**: only services with a legitimate operational requirement receive explicit permission through the firewall.

---

### 7. Explicitly Blocked High-Risk Services

Explicit `DROP` rules were created to document and enforce restrictions against several high-risk services:

```bash
sudo iptables -A INPUT -p tcp --dport 21 -j DROP
sudo iptables -A INPUT -p tcp --dport 23 -j DROP
sudo iptables -A INPUT -p tcp --dport 1524 -j DROP
```

These rules restricted:

| Port | Service | Action |
| --- | --- | --- |
| TCP 21 | FTP | DROP |
| TCP 23 | Telnet | DROP |
| TCP 1524 | Metasploitable Bindshell | DROP |
| TCP 22 | SSH | ACCEPT |

Although the default `INPUT DROP` policy already restricted unauthorized inbound traffic, explicitly defining these rules documented the intended security policy.

---

### 8. Reviewed the Final Firewall Ruleset

After configuring the firewall, I reviewed the resulting `iptables` rules using:

```bash
sudo iptables -L -n -v
```

<img width="1106" height="614" alt="image" src="https://github.com/user-attachments/assets/3a0006f6-c791-4994-85ca-5f87b713ba8a" />
)

**Figure 5 — Final iptables firewall ruleset.** The INPUT chain uses a default `DROP` policy, permits established connections and SSH traffic, and explicitly drops FTP, Telnet, and bindshell traffic.

The final configuration demonstrated several important firewall concepts:

- **Default deny** for incoming traffic
- **Stateful inspection** using `ESTABLISHED` and `RELATED`
- **SSH allowlisting** for secure administration
- **FTP blocking**
- **Telnet blocking**
- **Bindshell blocking**
- **Least-privilege network access**

The resulting firewall policy significantly reduced the number of services that could be accessed remotely.

---

## Firewall vs. Intrusion Prevention System

The lab also examined the differences between traditional firewall filtering and an **Intrusion Prevention System (IPS)**.

| Capability | Firewall | IPS |
| --- | --- | --- |
| Network Access Control | Primary Function | Secondary Function |
| Port/Protocol Filtering | Yes | Yes |
| Detect Known Exploits | Limited | Yes |
| Deep Packet Inspection | Limited | Yes |
| Detect Suspicious Behavior | Limited | Yes |
| Block Malicious Activity | Based on Rules | Based on Detection |

A firewall primarily controls **which network connections are permitted**, while an IPS analyzes permitted traffic for signs of malicious activity.

Examples of activity an IPS may identify include:

- Port scanning
- Brute-force authentication attempts
- Known exploit patterns
- Protocol violations
- Suspicious network behavior

This demonstrates why firewalls and intrusion prevention systems complement each other rather than serving as replacements for one another.

---

## Defense in Depth

This lab reinforced the importance of **defense in depth**, where multiple security controls work together to protect a system.

The **firewall** reduces the attack surface by restricting unnecessary network access.

The **IPS** provides additional analysis of permitted network traffic to identify potentially malicious behavior.

**Authentication** verifies users attempting to access authorized services.

**Privilege management** determines what authenticated users are permitted to do after gaining access.

Combining these controls creates multiple security barriers and reduces dependence on any single defensive technology.

---

## Security Analysis

The baseline assessment demonstrated how excessive service exposure can create a significant security risk.

Before firewall controls were implemented, the Metasploitable 2 system exposed **29 TCP services**, including legacy protocols, remote-access services, database services, and an insecure bindshell.

The firewall policy applied a **default-deny approach**, allowing only required traffic while restricting unnecessary services.

This reduced the externally accessible attack surface and demonstrated how properly designed firewall rules can enforce:

- Least privilege
- Least functionality
- Network access control
- Stateful inspection
- Secure remote administration

The lab also demonstrated that firewall protection alone does not eliminate security risk. Services permitted through the firewall may still contain vulnerabilities or be targeted through authentication attacks.

For this reason, firewall controls should operate alongside IPS, authentication, privilege management, endpoint protection, logging, and monitoring.

---

## Key Security Recommendations

- Implement default-deny firewall policies where appropriate.
- Expose only services required for legitimate operations.
- Replace insecure protocols such as Telnet with encrypted alternatives such as SSH.
- Restrict administrative services to authorized systems and networks.
- Regularly review and audit firewall rules.
- Use stateful inspection to control connection traffic.
- Remove or disable unnecessary services.
- Deploy host-based firewalls alongside network perimeter controls.
- Use IPS technology to analyze malicious activity within permitted traffic.
- Implement strong authentication for administrative services.
- Apply least privilege to user and administrative accounts.
- Combine multiple security controls to establish defense in depth.

---

## Conclusion

This project provided hands-on experience implementing and analyzing **host-based firewall security controls** in a controlled cybersecurity lab environment.

By performing network reconnaissance with Nmap, identifying high-risk services, establishing secure SSH administration, implementing a default-deny `iptables` policy, configuring stateful inspection, and explicitly restricting dangerous services, I gained practical experience with network access control and attack surface reduction.

The project also strengthened my understanding of how **firewalls, intrusion prevention systems, authentication, and privilege management work together as part of a defense-in-depth security strategy**.
