# Campus Network Project: Team Guide

**Course:** Computer Networks · **Tool:** Cisco Packet Tracer · **File:** `CampusManagementSystem.pkt`

## 1. What are we building?

We built a **campus network split into separate segments** (staff, Engineering, Placement, and two labs). Each segment has its own IP subnet, and **routers** connect them so that a PC in one segment can reach a PC in another. Inside the labs we also used **VLANs** so that Lab 1 and Lab 2 share one physical switch but stay separate networks.

A second, related diagram shows a **firewall router with an ACL** (access control list) that allows some traffic and blocks other traffic.

**In one sentence:** staff, Engineering, Placement and lab subnets are connected through routers, static routes and VLANs, and a related firewall diagram illustrates ACL-based traffic control.

## 2. The big picture

Read the network from top to bottom:

```text
Staff PCs (192.168.10.0/24)
   |
Management_switch --> Management router (gateway 192.168.10.1)
   |
Core_Switch  ----- Core transit network 10.0.0.0/24
   |                (Management 10.0.0.1, Engineering 10.0.0.6)
   |
LJ_Engineering_department router
   |
Engineering transit network 10.0.1.0/24
(Engineering 10.0.1.1)
   |                         |
Placement_Router (10.0.1.2)  Lab router (10.0.1.3)
   |                         |
Placement PCs                LABS_Core_Switch (VLANs 30 and 31)
192.168.20.0/24              |            |
                          Lab 1         Lab 2
                       192.168.30.0/24  192.168.31.0/24
```

The BSC, IMBA, Pharmacy and Law routers also appear in the topology, but our configuration and testing focused on **Engineering, Placement and the Labs**.

## 3. IP plan (the reference table)

| Segment | What is in it | Network | Key addresses |
| --- | --- | --- | --- |
| Staff | `Admin_PC`, `Higher_HR_PC`, `Finance_handler_PC` → `Management_switch` → `Management_Staff` router | `192.168.10.0/24` | Gateway `192.168.10.1` |
| Core transit | `Core_Switch` linking management and department routers | `10.0.0.0/24` | Management `10.0.0.1`, Engineering `10.0.0.6` |
| Engineering transit | `LJ_Engineering_department` linking Placement and Labs | `10.0.1.0/24` | Engineering `10.0.1.1`, Placement router `10.0.1.2`, Lab router `10.0.1.3` |
| Placement | 3 PCs → switch → `Placement_Router` | `192.168.20.0/24` | Gateway `.1`, PCs `.10` to `.12` |
| Lab 1 | 3 PCs → hub → lab switch/router | VLAN 30 · `192.168.30.0/24` | Gateway `192.168.30.1`, PCs `.10` to `.12` |
| Lab 2 | 3 PCs → `Lab2_Switch` → lab switch/router | VLAN 31 · `192.168.31.0/24` | Gateway `192.168.31.1`, PCs `.10` to `.12` |

**Two kinds of networks:** the `192.168.x.0` networks are where PCs live (LANs). The `10.0.x.0` networks are *transit links* that exist only to connect routers to each other.

## 4. How a ping travels (worked examples)

Understanding one full path makes the whole project click.

**Example A: Placement PC to a Lab 1 PC**

1. The Placement PC sees the destination is **not** in `192.168.20.0/24`, so it sends the packet to its **default gateway** (`192.168.20.1`, the Placement router).
2. The Placement router checks its routing table for `192.168.30.0/24`. A static route sends it toward the Lab router (or via Engineering, depending on how the route was entered; confirm with `show ip route`).
3. The Lab router receives it on subinterface **G0/0.30** (VLAN 30) and delivers it to the Lab 1 PC.
4. The reply comes back the same way in reverse. **Both directions need routes**, otherwise the ping fails even though the forward path works.

**Example B: Admin PC to Placement PC** (expected path)

Admin PC → Management router (`192.168.10.1`) → core transit `10.0.0.0/24` → Engineering router → transit `10.0.1.0/24` → Placement router → Placement PC.

**Example C: Lab 1 to Lab 2**

Both labs hang off one physical link to the Lab router. Traffic goes up on VLAN 30, the router moves it between its two subinterfaces, and it goes back down on VLAN 31. This is **router-on-a-stick**.

## 5. What is configured

- Physical connections between staff, core, Engineering, Placement and Labs.
- **VLAN 30 (`LAB1`)** and **VLAN 31 (`LAB2`)** on `LABS_Core_Switch`.
- Lab **access ports** (one VLAN each) and a **trunk** carrying VLANs 30 and 31.
- **Router-on-a-stick** on `Lab_Router`: `G0/0.30` = `192.168.30.1`, `G0/0.31` = `192.168.31.1`.
- **Static routes** adjusted on the Management, Engineering and Placement routers.
- Project saved as `CampusManagementSystem.pkt`.

## 6. What has been tested

Reported as successful:

- Lab 1 ↔ Lab 2
- Placement PC → Lab 1 and Lab 2 PCs
- Placement PC ↔ Admin PC
- Admin PC → sample hosts in Placement, Lab 1 and Lab 2

**Be honest about the limits:** these are sample tests. They do not prove that every department or every host pair works (for example the BSC, IMBA, Pharmacy and Law routers were not the focus).

## 7. Key concepts in plain language

| Term | Meaning | Where we use it |
| --- | --- | --- |
| **Switch** | Connects devices inside one LAN | Management, Placement and Lab switches |
| **Router** | Connects different IP networks | Management, Engineering, Placement, Lab routers |
| **Default gateway** | The router a PC sends traffic to when the destination is outside its own subnet | `.1` address in each LAN |
| **VLAN** | Splits one physical switch into separate Layer 2 networks | VLAN 30 and 31 |
| **Access port** | Switch port that belongs to one VLAN | Ports facing lab PCs |
| **Trunk** | Link that carries several VLANs at once | Switch to Lab router |
| **Router-on-a-stick** | One router link with a subinterface per VLAN, routing between VLANs | `G0/0.30` and `G0/0.31` |
| **Static route** | A manually entered path to a remote subnet | Management, Engineering, Placement routers |
| **ACL** | Ordered permit/deny rules that match traffic | Firewall diagram |

## 8. Firewall / ACL diagram

This is a related scenario:

- Employee LAN: `192.168.1.0/24`
- Admin LAN: `192.168.2.0/24`
- Private server LAN: `192.168.3.0/24`
- The Employee and Admin gateways connect to `Firewall_Router`, and `Private_Server` also connects to it.

**Caution:** the diagram alone does not prove ACL 101 is configured. If it is part of our submission, we must verify three things: the ACL exists, it is applied to the right interface and direction, and one allowed flow and one blocked flow behave as expected.

**Open question for the team:** is this a separate scenario or part of the same deliverable? Confirm before the demo.

## 9. Demo cheat sheet

### Commands and what to look for

| Device | Command | What it shows |
| --- | --- | --- |
| Router | `show ip interface brief` | Interfaces are **up/up** with the correct IPs |
| Router | `show ip route` | Connected (`C`) and static (`S`) routes to remote subnets |
| Router | `show running-config` | Full configuration, including subinterfaces and routes |
| Lab switch | `show vlan brief` | VLAN 30 and 31 exist with the right ports |
| Lab switch | `show interfaces trunk` | The trunk is up and allows VLANs 30 and 31 |
| Firewall (if configured) | `show access-lists` | ACL rules and hit counters |
| PC (Desktop → Command Prompt) | `ipconfig` | IP, mask and gateway are correct |
| PC | `ping <destination-IP>` | End-to-end connectivity |

### 2-minute demo order

1. Explain the separate campus subnets.
2. Show the core → Engineering → Placement/Labs topology.
3. Show VLAN 30/31 and the lab trunk.
4. Show router interfaces and static routes.
5. Ping from Placement to Lab 1 and Lab 2.
6. If presenting ACLs, show the ACL and demonstrate allowed vs. blocked traffic.

**Tip:** the first ping in Packet Tracer often times out while ARP resolves. Run it twice before concluding it failed.

## 10. Likely viva questions, with answers

**Why separate subnets?** It keeps each department's traffic contained, makes the network easier to manage and secure, and forces inter-segment traffic through routers where we can control it.

**Why VLANs 30 and 31?** They keep Lab 1 and Lab 2 as separate Layer 2 networks even though they share the same lab switch.

**Why a trunk?** The trunk carries both VLANs over one physical link to the router, so we do not need one cable per VLAN.

**Why a default gateway?** A PC only knows how to deliver locally. For any destination outside its subnet it needs a router to forward the packet.

**Why static routes?** Routers only know their directly connected networks. Static routes tell them how to reach remote subnets. They are simple and predictable, but do not adapt automatically if a link fails (dynamic routing such as OSPF would).

**Why can a ping fail even when all links are green?** Green links only mean the physical layer is up. The cause could be a wrong IP, mask or gateway, wrong VLAN membership, a missing route (in either direction), or an ACL blocking the traffic.

**How do you prove an ACL works?** Show its configuration and where it is applied, then test one allowed and one denied flow, and show the match counters in `show access-lists`.

## 11. Troubleshooting checklist

If a ping fails, work from the bottom up:

1. **PC:** run `ipconfig`. Are the IP, mask and gateway right?
2. **Gateway:** can the PC ping its own gateway? If not, check the cable, switch port or VLAN.
3. **Router interfaces:** `show ip interface brief`. Is everything up/up with the right IP?
4. **Routes:** `show ip route`. Is there a route to the destination subnet **and** a return route on the far side?
5. **VLANs (labs):** `show vlan brief` and `show interfaces trunk`. Are ports in the right VLAN and does the trunk allow 30/31?
6. **ACLs:** `show access-lists`. Is something denying the traffic?

## 12. Final checks before presenting

- [ ] Save `CampusManagementSystem.pkt`.
- [ ] Verify IPs, masks, gateways, router interfaces and routes.
- [ ] Confirm VLAN membership and trunk status.
- [ ] Repeat the key pings right before presenting.
- [ ] Decide whether the firewall/ACL diagram is a separate scenario or part of this deliverable.
- [ ] Make sure every teammate can explain one full ping path (Section 4) in their own words.

## 13. Items to double-check

These points came from the original summary and are worth confirming in the `.pkt` file so nobody gets caught out in the viva:

- Lab 1 is described as using a **hub** while Lab 2 uses a **switch**. Be ready to explain why (a hub is a Layer 1 device, so it does not understand VLANs).
- The exact next hop used in each static route (confirm with `show ip route`).
- Whether the other department routers (BSC, IMBA, Pharmacy, Law) are configured or only placed in the topology.
