# Configuration Walkthrough

This document walks through the full configuration of the retail store network lab, device by device, with annotated screenshots taken directly from Cisco Packet Tracer.

## Topology

![Topology](./Images/TOPOLOGY.png)

The store network is split into an **edge layer** (edge-R1 ↔ ISP), a **core L3 switch** (L3-SW1) that owns all inter-VLAN routing and DHCP, and two **access switches** (SW1, SW2) that connect the end devices.

---

## 1. VLAN Configuration

VLANs were created consistently across all three switches so that traffic stays segmented from the access layer all the way up to the core.

**L3-SW1 — VLAN table**
![VLAN L3-SW1](./Images/01,VLAN,L3-SW1.png)

**SW1 — VLAN table**
![VLAN SW1](./Images/02,VLAN,SW1.png)

**SW2 — VLAN table**
![VLAN SW2](./Images/03,VLAN,SW2.png)

| VLAN | Name | Description |
|------|------|-------------|
| 10 | POS | Point-of-sale registers |
| 20 | SELF_CHECKOUT | Self-checkout kiosks |
| 30 | BACK_OFFICE | Manager & inventory PCs |
| 40 | GUEST_WIFI | Guest wireless access point |
| 50 | PRINTERS | Network printer(s) |
| 99 | NETWORK_MANAGEMENT | Switch/router management |
| 999 | UNUSED | Unused ports parking VLAN |

---

## 2. Access Layer — Port Security & Edge Protection

Every access port is hardened with **PortFast** (immediate forwarding, no STP delay) and **BPDU Guard** (shuts the port down if it receives a BPDU, protecting against rogue switches). Ports facing fixed, known devices additionally use **port security with sticky MAC** learning to lock the port to the first MAC address seen.

**SW1 — Fa0/1–Fa0/4, POS registers (VLAN 10, port security)**
![Access VLAN10 SW1](./Images/04,ACCESSPORT,VLAN10,PORT-SECURITY,PORTFAST,BPDUGUARD,SW1.png)

**SW1 — Fa0/5–Fa0/7, self-checkout kiosks (VLAN 20, port security)**
![Access VLAN10 SW1 pt2](./Images/05,ACCESSPORT,VLAN10,PORT-SECURITY,PORTFAST,BPDUGUARD,SW1.png)

**SW1 — Fa0/8, guest Wi-Fi access point (VLAN 40)**
![Access VLAN40 SW1](./Images/06,ACCESSPORT,VLAN40,PORTFAST,BPDUGUARD,SW1.png)

**SW2 — Fa0/1–Fa0/2, back office PCs (VLAN 30)**
![Access VLAN30 SW2](./Images/07,ACCESSPORT,VLAN30,PORTFAST,BPDUGUARD,SW2.png)

**SW2 — Fa0/3, network printer (VLAN 50)**
![Access VLAN50 SW2](./Images/08,ACCESSPORT,VLAN50,PORTFAST,BPDUGUARD,SW2.png)

---

## 3. Trunk Links

SW1 and SW2 each trunk a single uplink back to the core (L3-SW1), carrying only the VLANs actually needed on that switch — nothing more.

**SW1 uplink trunk**
![Trunk SW1](./Images/09,TRUNK,SW1.png)

**SW2 uplink trunk**
![Trunk SW2](./Images/10,TRUNK,SW2.png)

**L3-SW1 — both trunks (Gig0/1 to SW1, Gig0/2 to SW2)**
![Trunk L3-SW1](./Images/11,TRUNK,L3-SW1.png)

---

## 4. Layer 3 — SVIs & Inter-VLAN Routing

L3-SW1 hosts one SVI per VLAN, acting as the default gateway for every subnet in the store.

![SVIs L3-SW1](./Images/12,SVI's,L3-SW1.png)

---

## 5. DHCP

L3-SW1 also runs DHCP for every internal VLAN, with the gateway address in each subnet excluded from the pool.

![DHCP L3-SW1](./Images/13,DHCP,L3-SW1.png)

---

## 6. Switch Management

All three switches are managed out of a dedicated **VLAN 99 (NETWORK_MANAGEMENT)**, reachable only over **SSH** (Telnet is disabled on the VTY lines).

**SW1 — management SVI**
![Mgmt VLAN SW1](./Images/14,SW-MANAGEMENT-VLAN,SW1.png)

**SW2 — management SVI**
![Mgmt VLAN SW2](./Images/15,SW-MANAGEMENT-VLAN,SW2.png)

**SW1 — SSH-only VTY lines**
![SSH SW1](./Images/16,SSH,SW1.png)

**SW2 — SSH-only VTY lines**
![SSH SW2](./Images/17,SSH,SW2.png)

**L3-SW1 — SSH-only VTY lines**
![SSH L3-SW1](./Images/18,SSH,L3-SW1.png)

---

## 7. Core-to-Edge Link

L3-SW1 connects to edge-R1 over a dedicated, routed (non-switchport) point-to-point link.

**L3-SW1 — routed port to edge-R1**
![Routed Port L3-SW1](./Images/19,ROUTED-PORT,L3-SW1.png)

---

## 8. Edge Router (edge-R1)

edge-R1 sits between the store network and the ISP, translating all internal traffic via **NAT overload (PAT)** on its outside interface.

**Interfaces — Gi0/0 (to ISP, NAT outside) & Gi0/1 (to L3-SW1, NAT inside)**
![Interfaces edge-R1](./Images/20,INTERFACES,EDGE-R1.png)

**NAT configuration — ACL matching internal 192.168.0.0/16 traffic, overloaded out Gi0/0**
![NAT edge-R1](./Images/22,NAT,EDGE-R1.png)

---

## 9. Routing

**edge-R1 — routing table** (default route to the ISP, static route back to the internal 192.168.0.0/16 range via L3-SW1)
![Routes edge-R1](./Images/21,ROUTES,EDGE-R1.png)

**L3-SW1 — routing table** (all VLANs directly connected, default route out to edge-R1)
![Routes L3-SW1](./Images/23,ROUTES,L3-SW1.png)

---

## Summary

| Layer | Device | Role |
|-------|--------|------|
| Edge | edge-R1 | NAT/PAT, default route to ISP, static route back to LAN |
| Core (L3) | L3-SW1 | Inter-VLAN routing, DHCP, SVIs, trunk aggregation |
| Access | SW1, SW2 | VLAN assignment, port security, PortFast/BPDU Guard, trunking to core |
| Management | VLAN 99 | SSH-only, out-of-band-style access to every switch |
