# pfSense Firewall Project


## Overview

This lab involved installing and configuring pfSense in a Hyper-V virtual environment.

The objective was to create a segmented network environment containing an Internal Company Network, DMZ Network, Security Testing Network and INTERNET/WAN in order to configure and test firewall rules controlling traffic between these networks.

## Key Learnings/Skills Learned

- Learned how to configure pfSense as a virtual firewall.
- Learned how separate virtual networks can be connected through pfSense.
- Learned how firewall rules control traffic between network segments.
- Learned that rule order is important when multiple rules could match the same traffic.
- Learned how firewall logging can provide evidence of allowed and blocked traffic.

## Lab Environment

| Component | Purpose |
|---|---|
| pfSense | Firewall/router |
| Debian | HR network client |
| Metasploitable | Sa|
| Kali Linux |  |


## Network Topology

| Network | Subnet |
|---|---|
| Internal network | 10.0.0.0/24 |
| DMZ | 10.30.0.0/24 |
| Security Testing Network | 192.168.2.0/24 | 
| External network | 192.168.0.0/24 | 

### Network Diagram

![Network Diagram](screenshots/network-diagram.png)

## Task 1 – Hyper-V Networking

I created the required internal virtual switches for the BLUE, PURPLE and RED networks.

The virtual switches were configured as internal networks and connected to the appropriate pfSense interfaces.

### Evidence

![Hyper-V Virtual Switches](screenshots/01-network-switches.png)

**Result:** The required virtual networks were successfully created.

---

## Task 2 – Host Network Adapters

The Hyper-V virtual Ethernet adapters were assigned static IP addresses.

| Adapter | IP Address | Subnet Mask |
|---|---|---|
| BLUE | 192.168.1.5 | 255.255.255.0 |
| PURPLE | 10.30.0.5 | 255.255.255.0 |
| RED | 192.168.2.5 | 255.255.255.0 |

### Evidence

![Host Network Adapters](screenshots/02-host-adapters.png)

---

## Task 3 – DHCP Scopes

DHCP scopes were created for the BLUE, PURPLE and RED networks.

| Network | DHCP Range | Gateway |
|---|---|---|
| BLUE | 192.168.1.100–192.168.1.200 | 192.168.1.1 |
| PURPLE | 10.30.0.100–10.30.0.200 | 10.30.0.1 |
| RED | 192.168.2.100–192.168.2.200 | 192.168.2.1 |

### Evidence

![DHCP Scopes](screenshots/03-dhcp-scopes.png)

**Result:** The DHCP scopes were successfully created and activated.

---

## Task 4 – Virtual Machine Connections

The virtual machines were connected to their respective networks.

| VM | Network | Expected IP |
|---|---|---|
| Debian | BLUE | 192.168.1.100 |
| Metasploitable | PURPLE | 10.30.0.100 |
| Kali | RED | 192.168.2.100 |

### Evidence

![VM Network Configuration](screenshots/04-vm-network-settings.png)

---

## Task 5 – pfSense Installation and Configuration

A pfSense virtual machine was created in Hyper-V with four network adapters.

The adapters were connected to:

1. BLUE
2. PURPLE
3. RED
4. INTERNET/WAN

### pfSense Interfaces

| pfSense Interface | Network | IP Address |
|---|---|---|
| BLUELAN | BLUE | 192.168.1.1/24 |
| PURPLE_DMZ | PURPLE | 10.30.0.1/24 |
| REDLAN | RED | 192.168.2.1/24 |
| INTERNETWAN | WAN | DHCP |

### Evidence

![pfSense Interfaces](screenshots/05-pfsense-interfaces.png)

**Result:** pfSense was successfully installed and configured with four network interfaces.

---

# Task 6 – Firewall Rules

## BLUE Network Rules

Firewall rules were configured on the BLUE interface to control traffic between the internal network and the PURPLE/DMZ network.

### Evidence

![BLUE Firewall Rules](screenshots/06-blue-firewall-rules.png)

### Testing

I tested connectivity from the Debian VM to the Metasploitable VM using:

```bash
telnet <METASPLOITABLE-IP>
```

and HTTP access using a web browser.

### Result

The Telnet connection was blocked/time-out occurred while HTTP access remained available according to the configured firewall rules.

![BLUE Firewall Test](screenshots/07-blue-firewall-test.png)

---

## PURPLE Network Rules

The PURPLE network was configured with:

- A block rule preventing traffic from PURPLE to BLUE
- A pass rule allowing other traffic

The block rule was placed above the pass rule so that the specific blocked traffic was processed first.

### Evidence

![PURPLE Firewall Rules](screenshots/08-purple-firewall-rules.png)

### Testing

Connectivity was tested between the Metasploitable VM and the other networks.

**Expected result:**

- PURPLE → BLUE = blocked
- PURPLE → other permitted destinations = allowed

![PURPLE Firewall Test](screenshots/09-purple-firewall-test.png)

---

## RED Network Rules

The RED network was configured with:

- Block all IPv4 traffic from RED to BLUE
- Allow all IPv4 traffic from RED to other destinations

Firewall logging was enabled for these rules so that allowed and blocked traffic could be recorded.

### Evidence

![RED Firewall Rules](screenshots/10-red-firewall-rules.png)

---

## Firewall Testing Results

| Test | Expected Result | Actual Result |
|---|---|---|
| RED → BLUE | Blocked | [PASS/FAIL] |
| RED → PURPLE | Allowed | [PASS/FAIL] |
| PURPLE → BLUE | Blocked | [PASS/FAIL] |
| PURPLE → RED | Allowed | [PASS/FAIL] |
| BLUE → PURPLE | Allowed where rule permits | [PASS/FAIL] |

## Firewall Logs

Firewall logs were reviewed to confirm that traffic was being handled according to the configured rules.

![Firewall Logs](screenshots/11-firewall-logs.png)



## Conclusion

This lab demonstrated the configuration and testing of a pfSense firewall in a virtualised network environment. The firewall was used to control communication between internal, DMZ and attacker networks. Testing confirmed that the configured firewall rules could allow or block traffic according to the required security policy.
