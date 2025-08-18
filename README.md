# Open-Source-Project-for-SME-s

# Beginner’s Guide: A–Z OPNsense Home Firewall Setup

---

## Table of Contents

1. [Install1OPNsense](#install-opnsense)  
2. [Configure OPNsense](#configure-opnsense)  
   - Configuration Wizard  
   - System Settings  
   - Interface Configuration  
   - VLAN Setup  
   - Assign Interfaces  
   - Interface Pages  
   - DHCP Setup  
   - DNS Setup  
   - Firewall Rules & Aliases
  3. [Next Steps]


---

## 1. Install OPNsense

- Download OPNsense VGA installer ISO from the official site. Extract if required 
- Follow installation steps (e.g., on Protectli mini-PC or similar):
  1. Skip auto importer; manually assign interfaces  
  2. Skip LAGG and VLAN setup; do later in UI  
  3. Assign `vtnet0` as WAN, `vtnet1` as LAN (or your hardware equivalent)  
  4. Install using ZFS, set root password, complete install and reboot :contentReference[oaicite:5]{index=5}

---

## 3. Configure OPNsense

### Configuration Wizard
- Skip the wizard by clicking the OPNsense logo—this gives full access to all settings :contentReference[oaicite:6]{index=6}

### System Settings

**General (System → Settings → General)**  
| Option | Value |
|--------|-------|
| Hostname | `router` (or as you choose) |
| Domain | custom (e.g., `home.local`) |
| Time Zone | your local zone (not necessarily America/New_York) |
| DNS Servers | leave blank |
| Override DNS from DHCP on WAN | enabled :contentReference[oaicite:7]{index=7} |

**Administration (System → Settings → Administration)**  
- Protocol: HTTPS  
- Enable HTTP Strict Transport Security  
- Port: 443  
- Disable HTTP Redirect, DNS rebind checks  
- HTTP Compression: High (or Low for slower devices)  
- Listen on all interfaces :contentReference[oaicite:8]{index=8}

**Miscellaneous**  
- Thermal Sensors: CPU type  
- RRD Backup, DHCP Leases, NetFlow: every 24h (optional)  
- Use PowerD: checked; Power Mode: Hiadaptive :contentReference[oaicite:9]{index=9}

---

### Interface Configuration

**Settings (Interfaces → Settings)**  
- Disable hardware offloading: CRC, TSO, LRO  
- Disable VLAN hardware filtering  
- Recommended when using IDS/IPS or Zenarmor due to netmap incompatibility :contentReference[oaicite:10]{index=10}

**VLAN Setup (Interfaces → Other Types → VLAN)**  
- Parent: LAN (e.g., `igc1`)  
- VLAN Tag: 10  
- Description: UNTRUSTED  
- Device: leave blank (auto-generates) :contentReference[oaicite:11]{index=11}

**Assign Interfaces (Interfaces → Assignments)**  
- Click “+”, select the new UNTRUSTED VLAN  
- Give it the name `UNTRUSTED` (to avoid confusion with OPT1/OPT2) :contentReference[oaicite:12]{index=12}

**Interface Pages (Interfaces → [WAN], [LAN], [UNTRUSTED])**  
- **WAN Interface**: enable, lock (prevent removal), block private & bogon networks, IPv4: DHCP, IPv6: None :contentReference[oaicite:13]{index=13}  
- **LAN & UNTRUSTED**:
  - Enable interface, lock  
  - Uncheck block private/bogon  
  - IPv4 type: Static  
  - IPv6: None  
  - IPv4 upstream gateway: Auto-detect  
  - LAN: 192.168.1.1/24; UNTRUSTED: 192.168.10.1/24 :contentReference[oaicite:14]{index=14}

---

### DHCP Configuration (Services → DHCPv4)

- **LAN**: Enable, Range: 192.168.1.100–192.168.1.200  
- **UNTRUSTED**: Enable, Range: 192.168.10.100–192.168.10.200  
  *(Reserve addresses below range for static assignments)* :contentReference[oaicite:15]{index=15}

---

### DNS Configuration

**System Settings → General**  
- Leave DNS blank; allow override via DHCP/PPP from WAN :contentReference[oaicite:16]{index=16}

**Unbound DNS (Services → Unbound DNS → General)**  
- Enable Unbound  
- Listen on all interfaces  
- DNSSEC: enable if upstream supports  
- Register DHCP leases & static mappings  
- Flush cache on reload  
- Local zone type: transparent  
- Apply changes :contentReference[oaicite:17]{index=17}

---

### Firewall Configuration (Firewall → …)

**Aliases (Firewall → Aliases)**  
- Add alias `PrivateNetworks` of type Networks: include `10.0.0.0/8,172.16.0.0/12,192.168.0.0/16`  
  Simplifies rules to block internal networks :contentReference[oaicite:18]{index=18}

**LAN Rules**  
- Remove default “allow all” IPv4/IPv6 rules  
- Add, in this order:
  1. Allow DNS (TCP/UDP) from LAN net to LAN address port 53  
  2. Allow ICMPv4 from LAN net to any  
  3. Block access to `PrivateNetworks` (invert destination), allow internet  
  *(Note: Invert destination must be checked on rule 3)* :contentReference[oaicite:19]{index=19}

**UNTRUSTED Rules**  
- Allow DNS from UNTRUSTED net to UNTRUSTED address port 53  
- Block internal networks to PrivateNetworks (invert), allow internet only :contentReference[oaicite:20]{index=20}

---

## 3. Next Steps

- You now have a basic, secure dual-network setup with VLANs separating trusted/untrusted devices.
- Optional enhancements:
  - **Additional security**: e.g., IDS/IPS, captive portal :contentReference[oaicite:27]{index=27}  
  - **Multi-homing**: e.g., connect NAS to both networks using dual NICs for segregation and performance gains :contentReference[oaicite:28]{index=28}  
  - **Remote access**: configure WireGuard, OpenVPN, IPSec for secure external access :contentReference[oaicite:29]{index=29}  

---


## Summary of Key Points

- Installed OPNsense with manual interface assignment
- Configured system settings and disabled hardware offloading
- Created VLAN 10 for untrusted devices
- Assigned interfaces, configured static IPs and DHCP ranges
- Set up DNS via Unbound, firewall aliases, and strict network isolation rules
- Configured a VLANs on FW
- Achieved secure segmentation between trusted and untrusted devices


