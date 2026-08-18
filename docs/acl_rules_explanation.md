# Firewall Rules (ACLs) Explanation - In-Depth Analysis

> 📖 **Main Document:** For complete technical implementation (switch and router configuration, DHCP, VLANs, and detailed test results), see [README.md](../README.md)

## 📌 Introduction

Firewall Rules in this project are implemented using **Access Control Lists (ACLs)** on a Cisco Router. ACLs act as filters for network traffic based on defined rules. This configuration enforces **Zero Trust** principles where access between network segments is not granted by default.

## 🔐 Security Analysis: Why These Rules Matter

### Threats Mitigated (Threat Scenarios)

**Scenario 1: Insider Threat (Malicious Employee)**
- A disgruntled HR employee attempts to access IT servers to steal sensitive data.
- The ACL blocks access from VLAN 20 to VLAN 10, preventing unauthorized access.

**Scenario 2: Malware/Ransomware**
- An HR workstation is infected with ransomware that attempts to spread to IT servers.
- The ACL blocks lateral movement, isolating the attack to VLAN 20 only.

### Zero Trust Principles Applied

| Principle | Implementation |
|:---|:---|
| **"Never Trust, Always Verify"** | No default access between VLANs |
| **Least Privilege Access** | HR can only access required resources |
| **Micro-segmentation** | VLANs separate departments logically |

---

## 🔍 Rule Analysis

### Rule 1: Block HR → IT (ICMP Echo Request)

```
access-list 100 deny icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 echo
```

| Component | Value | Explanation |
|:---|:---|:---|
| **access-list** | 100 | ACL number (Extended ACL) - chosen because it can filter by source and destination IP |
| **deny** | - | Deny/block traffic |
| **icmp** | - | ICMP protocol (ping) |
| **192.168.20.0** | Source Network | HR network (VLAN 20) |
| **0.0.0.255** | Wildcard Mask | Represents /24 subnet |
| **192.168.10.0** | Destination Network | IT network (VLAN 10) |
| **echo** | - | ICMP Echo Request (ping initiation) |

**Purpose:** Block ping initiation from HR to IT.

---

### Rule 2: Allow HR → IT (ICMP Echo Reply)

```
access-list 100 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 echo-reply
```

**Purpose:** Allow ping replies (echo-reply) from HR to IT.

---

### Rule 3: Allow ALL Traffic from IT → HR

```
access-list 100 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
```

**Purpose:** Allow all traffic from IT to HR.

---

### Rule 4: Block ALL Traffic from HR → IT

```
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
```

**Purpose:** Block all traffic from HR to IT (except those explicitly allowed above).

---

### Rule 5: Allow All Other Traffic

```
access-list 100 permit ip any any
```

**Purpose:** Allow all other traffic.

---

## 🔧 Implementation

The ACL is applied to the **VLAN 20 sub-interface** with **inbound** direction:

```
interface GigabitEthernet0/0.20
 ip access-group 100 in
```

**Explanation:**
- **Sub-interface VLAN 20**: Traffic entering the router from VLAN 20 is checked.
- **"in" direction**: Traffic entering the interface (from HR to router) is filtered.
- **Effect**: Traffic from HR cannot reach other VLANs (except as allowed by Rule 2).

---

## ✅ Test Results

| Source | Destination | Result | Security Implication |
|:---|:---|:---:|:---|
| PC IT (VLAN 10) | Gateway HR | ✅ **SUCCESS** | IT can access HR for support |
| PC IT (VLAN 10) | Server HR | ✅ **SUCCESS** | IT can access Server HR |
| PC IT (VLAN 10) | PC HR | ✅ **SUCCESS** | IT can access HR |
| PC HR (VLAN 20) | Gateway IT | ❌ **DESTINATION UNREACHABLE** | **Prevents lateral movement!** |
| PC HR (VLAN 20) | PC IT | ❌ **DESTINATION UNREACHABLE** | **Prevents lateral movement!** |
| Server HR (VLAN 20) | Gateway IT | ❌ **DESTINATION UNREACHABLE** | ACL based on IP, not device! |
| PC3 (VLAN 30) | Gateway IT | ✅ **SUCCESS** | VLAN 30 without ACL! |
| PC3 (VLAN 30) | PC HR | ✅ **SUCCESS** | End devices CAN respond! |
| PC3 (VLAN 30) | PC IT | ✅ **SUCCESS** | End devices CAN respond! |
| **PC0 (VLAN 10)** | **PC3 (VLAN 30)** | ✅ **SUCCESS** | End devices CAN respond! |

---

## 💡 Why Use ACL Number 100?

| ACL Type | Range | Capability | Project Requirement |
|:---|:---:|:---|:---:|
| **Standard ACL** | 1-99 | Filter by source IP only | ❌ Not sufficient |
| **Extended ACL** | 100-199 | Filter by source IP, destination IP, and protocol | ✅ **Required** |

**Analysis:** Since we need to filter by both source **and** destination (HR → IT), we use an **Extended ACL (100)**. This provides more granular and precise control.

---

## 🎯 Final Conclusion

With this ACL implementation and supported by VLAN 30 validation, we successfully:

1. **Implemented Zero Trust**: No default access between VLANs
2. **Prevented Lateral Movement**: HR cannot access IT
3. **Reduced Attack Surface**: IT network is not exposed to HR
4. **Enforced Least Privilege**: HR can only access required resources

---

## 📚 Security References

- NIST SP 800-53: Access Control (AC) Family
- CIS Controls: Control 12 - Network Infrastructure Management
- Zero Trust Maturity Model (CISA)
