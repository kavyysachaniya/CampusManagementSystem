# CN Group Project Summary

**Computer Networks · Cisco Packet Tracer**

## Project

Build a segmented campus network and demonstrate inter-network routing.
A related diagram illustrates firewall ACL traffic filtering.

## Topology & IP Plan

  -----------------------------------------------------------------------------
  Segment                 Devices / purpose             Network / key IPs
  ----------------------- ----------------------------- -----------------------
  Staff                   `Admin_PC`, `Higher_HR_PC`,   `192.168.10.0/24`;
                          `Finance_handler_PC` →        gateway `192.168.10.1`
                          `Management_switch` →         
                          `Management_Staff`            

  Core transit            `Core_Switch` connects        `10.0.0.0/24`;
                          management and department     Management `10.0.0.1`,
                          routers                       Engineering `10.0.0.6`

  Engineering transit     `LJ_Engineering_department`   `10.0.1.0/24`;
                          connects Placement and Labs   Engineering `10.0.1.1`,
                                                        Placement router
                                                        `10.0.1.2`, Lab router
                                                        `10.0.1.3`

  Placement               3 PCs → switch →              `192.168.20.0/24`;
                          `Placement_Router`            gateway `.1`; PCs
                                                        `.10–.12`

  Lab 1                   3 PCs → hub → lab             VLAN 30 ·
                          switch/router                 `192.168.30.0/24`;
                                                        gateway `192.168.30.1`;
                                                        PCs `.10–.12`

  Lab 2                   3 PCs → `Lab2_Switch` → lab   VLAN 31 ·
                          switch/router                 `192.168.31.0/24`;
                                                        gateway `192.168.31.1`;
                                                        PCs `.10–.12`
  -----------------------------------------------------------------------------

**Note:** The BSC, IMBA, Pharmacy and Law routers are visible in the
topology, but the main configuration/testing focus was Engineering,
Placement and Labs.

## What's Configured

-   Physical connections between staff, core, Engineering, Placement and
    Labs.
-   VLAN 30 (`LAB1`) and VLAN 31 (`LAB2`) on `LABS_Core_Switch`.
-   Lab access ports and a trunk carrying VLANs 30/31.
-   Router-on-a-stick on `Lab_Router`: G0/0.30 → `192.168.30.1`; G0/0.31
    → `192.168.31.1`.
-   Static routes adjusted on Management, Engineering and Placement
    routers.
-   Project saved as `CampusManagementSystem.pkt`.

## Tested Successfully

-   Lab 1 ↔ Lab 2 connectivity.
-   Placement PC → Lab 1 and Lab 2 PCs.
-   Placement PC ↔ Admin PC.
-   Admin PC → Placement, Lab 1 and Lab 2 sample hosts.

These are reported successful tests; they do not prove every department
or every possible host pair works.

## Key Concepts

-   **Switch:** connects devices within a LAN.
-   **Router:** connects different IP networks.
-   **Default gateway:** next hop for traffic outside a PC's subnet.
-   **VLAN:** separates Layer 2 networks.
-   **Access port:** carries one VLAN; **trunk:** carries multiple
    VLANs.
-   **Router-on-a-stick:** router subinterfaces route between VLANs over
    one physical link.
-   **Static route:** manually defines a path to a remote subnet.
-   **ACL:** ordered permit/deny rules for matching traffic.

## Firewall / ACL Diagram

-   Employee LAN: `192.168.1.0/24`
-   Admin LAN: `192.168.2.0/24`
-   Private server LAN: `192.168.3.0/24`
-   Employee and Admin gateways connect to `Firewall_Router`;
    `Private_Server` also connects to it.

**Important:** The diagram alone does not prove ACL 101 is configured.
If this is part of the submission, verify the ACL, interface/direction,
and test one allowed and one blocked flow.

## Demo Commands

**Router**

``` text
show ip interface brief
show ip route
show running-config
```

**Lab switch**

``` text
show vlan brief
show interfaces trunk
```

**Firewall ACL (if configured)**

``` text
show access-lists
```

**PC:** Desktop → Command Prompt

``` text
ipconfig
ping <destination-IP>
```

## 2-Minute Demo Order

1.  Explain the separate campus subnets.
2.  Show the core → Engineering → Placement/Labs topology.
3.  Show VLAN 30/31 and the lab trunk.
4.  Show router interfaces and static routes.
5.  Ping from Placement to Lab 1 and Lab 2.
6.  If presenting ACLs, show the ACL and demonstrate allowed vs. blocked
    traffic.

## Likely Viva Questions

-   **Why separate subnets?** Easier management and routing between
    segments.
-   **Why VLANs 30 and 31?** Separate Lab 1 and Lab 2 Layer 2 networks.
-   **Why a trunk?** Carries both VLANs over one link.
-   **Why a default gateway?** Reaches destinations outside the local
    subnet.
-   **Why static routes?** Tell routers how to reach remote networks.
-   **Why can ping fail with green links?** IP, gateway, VLAN, routing
    or ACL configuration may still be wrong.
-   **How do you prove an ACL works?** Show its
    configuration/application and test allowed and denied traffic.

## Final Checks

-   [ ] Save `CampusManagementSystem.pkt`.
-   [ ] Verify IPs, masks, gateways, router interfaces and routes.
-   [ ] Confirm VLAN membership and trunk status.
-   [ ] Repeat the key pings before presenting.
-   [ ] Confirm whether the firewall/ACL diagram is a separate scenario
    or part of the same deliverable.

**One-line summary:** A Cisco Packet Tracer campus network connects
staff, Engineering, Placement and lab subnets using routers, static
routes and VLANs; a related firewall diagram illustrates ACL-based
traffic control.
