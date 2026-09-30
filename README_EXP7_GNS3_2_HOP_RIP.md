# Experiment 7 — Design a Simple 2-Hop Network and Configure RIP Using GNS3

## Objective

Design a simple **2-hop network** in GNS3, configure IPv4 addresses on the routers and PCs, configure **RIP version 2** for dynamic routing, verify the routing tables, test end-to-end connectivity, and capture packets using Wireshark.

> **What does 2-hop mean?**
> In this experiment, the end-to-end path crosses two routers: **R1 → R2 → R3**.

This README follows the topology and command sequence used during the successful setup and is intended to be reusable by another student.

---

## 1. Required Software / Components

- GNS3
- Cisco router images/templates supporting the serial interfaces used in the lab
- 3 Cisco routers: R1, R2, R3
- 2 Ethernet switches: Switch1, Switch2
- 2 VPCS hosts: PC1, PC2
- Wireshark

### Router serial interfaces

The setup used serial interfaces for the router-to-router links. The lab manual describes adding a serial module to the router and then using serial interfaces such as `s1/0`, `s1/1`, etc.

> **Important:** Your current GNS3 router model exposes interfaces as `Serial1/1`, `Serial1/2`, etc. Use the interface names that actually appear in `show ip interface brief` and in the GNS3 topology.

---

## 2. Final Topology

Use the following topology:

```text
                         R1                    R2                    R3
                       s1/1                  s1/1       s1/2      s1/1
                        |                       |----------|          |
                        |-----------------------|          |----------|
                        
                       f0/0                                          f0/0
                        |                                               |
                    Switch1                                           Switch2
                     /   \\                                             /   \
                   PC1   (LAN)                                         (LAN)  PC2
```

More clearly:

```text
PC1 --- Switch1 --- R1 ======== R2 ======== R3 --- Switch2 --- PC2
                       10.0.0.0/24   10.0.1.0/24
```

### Actual interface connections used

```text
R1 f0/0  <---->  Switch1
PC1 e0   <---->  Switch1

R1 s1/1  <---->  R2 s1/1
R2 s1/2  <---->  R3 s1/1

R3 f0/0  <---->  Switch2
PC2 e0   <---->  Switch2
```

### Screenshot 1 — Final GNS3 topology

> <img width="1518" height="677" alt="image" src="https://github.com/user-attachments/assets/5d8ed518-5f9b-426f-aa5f-7279a9dd53a8" />

>
> complete topology showing R1–R2–R3, both switches, both PCs, and green/active links.

<br><br><br>

---

## 3. IP Address Plan

Use three different IPv4 networks:

| Device | Interface | IPv4 Address | Subnet Mask | Purpose |
|---|---|---|---|---|
| R1 | f0/0 | `192.168.1.1` | `255.255.255.0` | Left LAN |
| PC1 | e0 | `192.168.1.2/24` | `255.255.255.0` | Left host |
| R1 | s1/1 | `10.0.0.1` | `255.255.255.0` | R1–R2 link |
| R2 | s1/1 | `10.0.0.2` | `255.255.255.0` | R1–R2 link |
| R2 | s1/2 | `10.0.1.1` | `255.255.255.0` | R2–R3 link |
| R3 | s1/1 | `10.0.1.2` | `255.255.255.0` | R2–R3 link |
| R3 | f0/0 | `192.168.2.1` | `255.255.255.0` | Right LAN |
| PC2 | e0 | `192.168.2.2/24` | `255.255.255.0` | Right host |

### Network summary

```text
192.168.1.0/24  -> Left LAN
10.0.0.0/24     -> R1 to R2
10.0.1.0/24     -> R2 to R3
192.168.2.0/24  -> Right LAN
```

---

## 4. Before Configuration — Check Interface Names

Open each router and run:

```text
show ip interface brief
```

Make sure the connected interfaces match your topology.

For this setup, the important interfaces are:

```text
R1: f0/0, s1/1
R2: s1/1, s1/2
R3: s1/1, f0/0
```

### Screenshot 2 — Interface check

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/fa62d57e-997e-41c2-9852-a1fe275f2230" />

><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/f3cd235f-660a-4956-aa6a-f02475e48e35" />

> one or more router consoles showing `show ip interface brief`.

<br><br><br>

---

## 5. Configure R1

Open R1 console.

```text
enable
configure terminal

interface f0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

interface s1/1
ip address 10.0.0.1 255.255.255.0
no shutdown
exit

end
```

Check the result:

```text
show ip interface brief
```

Expected important entries:

```text
FastEthernet0/0   192.168.1.1   up   up
Serial1/1         10.0.0.1      up   up
```

### Screenshot 3 — R1 configuration

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/4f127145-d620-439e-87ea-46941751249b" />

><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/74745029-e627-4723-869c-f41a5bb8f503" />


<br><br><br>

---

## 6. Configure R2

R2 is the middle router, so it needs **two serial interfaces**.

```text
enable
configure terminal

interface s1/1
ip address 10.0.0.2 255.255.255.0
no shutdown
exit

interface s1/2
ip address 10.0.1.1 255.255.255.0
no shutdown
exit

end
```

Verify:

```text
show ip interface brief
```

Expected important entries:

```text
Serial1/1   10.0.0.2   up   up
Serial1/2   10.0.1.1   up   up
```

### Screenshot 4 — R2 configuration

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/fba82f12-09a1-4ab8-b446-af35f8b41655" />
><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/6536a0c7-717e-4093-9bbe-6da7a68b9d22" />


<br><br><br>

---

## 7. Configure R3

```text
enable
configure terminal

interface s1/1
ip address 10.0.1.2 255.255.255.0
no shutdown
exit

interface f0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
exit

end
```

Verify:

```text
show ip interface brief
```

Expected important entries:

```text
Serial1/1       10.0.1.2   up   up
FastEthernet0/0 192.168.2.1 up  up
```

### Screenshot 5 — R3 configuration

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/d003e1aa-babe-4d47-8575-75e118ee6226" />


<br><br><br>

---

## 8. Configure PC1

Open PC1 and enter:

```text
ip 192.168.1.2/24 192.168.1.1
```

Verify:

```text
show ip
```

### Screenshot 6 — PC1 configuration

> <img width="1600" height="713" alt="image" src="https://github.com/user-attachments/assets/6d9a948a-62ea-487f-b190-3000f575abe4" />


<br><br><br>

---

## 9. Configure PC2

Open PC2 and enter:

```text
ip 192.168.2.2/24 192.168.2.1
```

Verify:

```text
show ip
```

### Screenshot 7 — PC2 configuration

> <img width="1417" height="550" alt="image" src="https://github.com/user-attachments/assets/00e2ee48-c24f-49d2-8334-2ba31d91e5f7" />


<br><br><br>

---

## 10. Test the Direct Router-to-Router Links

Do this **before configuring RIP**. It makes troubleshooting much easier.

### R1 → R2

On R1:

```text
ping 10.0.0.2
```

Expected: successful replies.

### R2 → R3

On R2:

```text
ping 10.0.1.2
```

Expected: successful replies.

### PC1 → R1

On PC1:

```text
ping 192.168.1.1
```

### PC2 → R3

On PC2:

```text
ping 192.168.2.1
```

### Screenshot 8 — Direct-link ping tests

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/0282e01e-94df-4e92-bc20-b2325c0da145" />

><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/75427353-a7b1-4a6e-87e9-d124a6c51d6c" />

>  successful pings for R1→R2 and R2→R3.

<br><br><br>

---

## 11. Configure RIP Version 2

RIP is the dynamic routing protocol used in the lab manual. The manual specifically uses RIP version 2 and asks each router to advertise the networks directly connected to it.

### R1

```text
enable
configure terminal
router rip
version 2
network 192.168.1.0
network 10.0.0.0
end
```

### R2

```text
enable
configure terminal
router rip
version 2
network 10.0.0.0
network 10.0.1.0
end
```

### R3

```text
enable
configure terminal
router rip
version 2
network 10.0.1.0
network 192.168.2.0
end
```

### Important rule

Only advertise the networks **directly connected to that router**.

So:

```text
R1 -> 192.168.1.0 + 10.0.0.0
R2 -> 10.0.0.0 + 10.0.1.0
R3 -> 10.0.1.0 + 192.168.2.0
```

### Screenshot 9 — RIP configuration

> <img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/ce472273-b6e6-4c42-b27e-6562df14088b" />

><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/11ff789f-fc58-4ec7-a7ee-8c1149a60be4" />

><img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/53522faf-3347-4afb-889f-5d0378ae3229" />


>  R1, R2 and R3 showing `router rip`, `version 2`, and their `network` commands.

<br><br><br>

---

## 12. Verify RIP Routing Tables

On R1:

```text
show ip route
```

On R2:

```text
show ip route
```

On R3:

```text
show ip route
```

Routes learned through RIP are marked with:

```text
R
```

### What you should expect

After RIP converges:

**R1** should learn the remote networks toward the R3 side, including `10.0.1.0/24` and `192.168.2.0/24`.

**R2** should learn the two LAN networks, including `192.168.1.0/24` and `192.168.2.0/24`.

**R3** should learn the remote networks toward R1, including `10.0.0.0/24` and `192.168.1.0/24`.


<br><br><br>

---

## 13. End-to-End Ping Test

This is the most important connectivity test for the 2-hop experiment.

From **PC1**:

```text
ping 192.168.2.2
```

The expected path is:

```text
PC1
  ↓
Switch1
  ↓
R1
  ↓
R2
  ↓
R3
  ↓
Switch2
  ↓
PC2
```

A successful ping confirms that routing between the two LANs is working.

### Screenshot 13 — PC1 to PC2 successful ping

> <img width="1600" height="713" alt="image" src="https://github.com/user-attachments/assets/8a632586-1e6f-402f-9bcd-7310878933fc" />

>
>  PC1 terminal showing replies from `192.168.2.2`.

<br><br><br>

---

## 14. Wireshark Capture

The lab manual asks for packet capture and analysis.

### Start the capture

1. In GNS3, right-click the required link.
2. Choose **Start capture**.
3. Open Wireshark.
4. Run:

```text
ping 192.168.2.2
```

5. Stop the capture after enough packets are visible.

### Useful filters

ICMP ping traffic:

```text
icmp
```

RIP traffic (when visible in the capture):

```text
rip
```

### What to identify for the ping

- Source IP address
- Destination IP address
- ICMP Echo Request
- ICMP Echo Reply
- Number of packets
- Any packet loss

### Screenshot 14 — Wireshark ping capture

> <img width="1600" height="633" alt="image" src="https://github.com/user-attachments/assets/7fa07043-7f91-442b-931b-c7e83ec925ca" />


<br><br><br>

### Screenshot 15 — Wireshark packet details

> <img width="1600" height="569" alt="image" src="https://github.com/user-attachments/assets/309809bb-fcb7-4db3-b836-6305a844fd30" />

>
>expand one ICMP packet and show the protocol information.

<br><br><br>

---


---

## 14A. Actual Wireshark Captures from the Experiment

The following two captures are from the actual Experiment 7 run and can be kept as evidence in the lab record. They show both **RIP control traffic** and **ICMP/ping traffic**.

### Capture A — PC1 Ethernet interface

![Wireshark capture on PC1 Ethernet0](./c598e100-fedb-4a55-9859-6b499f638503.png)

**What this capture shows:**

- The capture was taken on the **PC1 Ethernet0 → Switch1 Ethernet1** link.
- RIP version 2 responses are visible from `192.168.1.1` to multicast address `224.0.0.9`.
- The selected RIPv2 packet shows **UDP source port 520 and destination port 520**.
- ARP traffic is visible while resolving the local gateway MAC address.
- ICMP Echo Requests are visible from `192.168.1.2` to `192.168.2.2`.
- In this particular capture, the router returned **ICMP Destination Unreachable (Host Unreachable)** for the PC1→PC2 ping attempts. This means the screenshot is useful as a **diagnostic capture**, but it should **not** be presented as proof of successful end-to-end connectivity.

**Lab-record observation:**

> RIP control packets and ICMP packets were observed on the PC1 LAN. RIPv2 used UDP port 520, and the captured ping attempt generated ICMP Echo Requests followed by Host Unreachable messages from the gateway.

---

### Capture B — R1 serial interface / R1–R2 link

![Wireshark capture on R1 serial interface](./f957e1bb-4f06-4d62-87f7-ebbbb20d647a.png)

**What this capture shows:**

- The capture contains traffic on the **R1–R2 serial link**.
- RIPv2 responses are visible between `10.0.0.1` and `10.0.0.2`, sent to multicast `224.0.0.9`.
- **SLARP** packets are visible as serial-link control/keepalive traffic.
- **CDP** packets are also visible, identifying the neighboring Cisco router and serial port information.
- The presence of repeated RIPv2 responses demonstrates that the routers are exchanging routing information over the serial link.

**Lab-record observation:**

> The R1–R2 serial link carried RIPv2 routing updates between `10.0.0.1` and `10.0.0.2`. SLARP and CDP control packets were also observed on the serial connection.

---

### Screenshot placement in the record

For the lab record, place **Capture A** under the PC/LAN packet-capture observation and **Capture B** under the serial-interface packet-capture observation required by the manual. The manual specifically asks students to run Wireshark on the serial interface and show a packet-capture screenshot.

### Important distinction

The two captures demonstrate different parts of the experiment:

```text
Capture A → PC1 LAN traffic
           → ARP
           → RIPv2
           → ICMP ping / destination-unreachable response

Capture B → R1–R2 serial traffic
           → RIPv2 routing updates
           → SLARP
           → CDP
```

The final end-to-end test should be recorded separately with a **successful** `ping 192.168.2.2` from PC1 after RIP convergence.

## 15. Analysis / Viva Notes

### Why are different subnets used?

Each Layer-3 link/network needs its own IP subnet:

```text
192.168.1.0/24  -> left LAN
10.0.0.0/24     -> R1-R2
10.0.1.0/24     -> R2-R3
192.168.2.0/24  -> right LAN
```

### Why is RIP needed?

R1 directly knows `192.168.1.0/24` and `10.0.0.0/24`.

It does not initially know how to reach `192.168.2.0/24`.

RIP lets the routers exchange route information so that the remote networks become reachable.

### What does `R` mean in `show ip route`?

`R` means the route was learned using RIP.

### Why use `no shutdown`?

Cisco router interfaces are administratively disabled by default in the lab setup. `no shutdown` enables them.

---

## 16. Troubleshooting

### Problem 1 — Serial interface is down

Run:

```text
show ip interface brief
```

The connected serial interfaces should be `up/up`.

Check both ends of the serial link and confirm `no shutdown` was entered on both.

### Problem 2 — R1 can ping R2, but PC1 cannot ping PC2

Check:

```text
R1: show ip route
R2: show ip route
R3: show ip route
```

Look for RIP-learned (`R`) routes to the remote LANs.

### Problem 3 — PC1 can reach R1 but not PC2

Check PC1:

```text
show ip
```

It should be:

```text
IP      = 192.168.1.2/24
Gateway = 192.168.1.1
```

Check PC2:

```text
show ip
```

It should be:

```text
IP      = 192.168.2.2/24
Gateway = 192.168.2.1
```

### Problem 4 — RIP routes are missing

Recheck the RIP configuration:

```text
router rip
version 2
network <directly-connected-network>
```

For this setup:

```text
R1 -> 192.168.1.0, 10.0.0.0
R2 -> 10.0.0.0, 10.0.1.0
R3 -> 10.0.1.0, 192.168.2.0
```

Wait briefly for RIP to converge, then run `show ip route` again.

### Problem 5 — A link looks correct but the command does not work

Do not assume an interface name. Run:

```text
show ip interface brief
```

and use the exact interface shown by your router template.

---

## 17. Final Results Checklist

```text
[ ] R1-R2 serial link is up/up
[ ] R2-R3 serial link is up/up
[ ] R1 LAN interface is up/up
[ ] R3 LAN interface is up/up
[ ] PC1 configured as 192.168.1.2/24
[ ] PC2 configured as 192.168.2.2/24
[ ] R1 configured for RIP v2
[ ] R2 configured for RIP v2
[ ] R3 configured for RIP v2
[ ] R1 routing table contains RIP routes
[ ] R2 routing table contains RIP routes
[ ] R3 routing table contains RIP routes
[ ] PC1 can ping PC2
[ ] Wireshark capture completed
[ ] Required screenshots saved
```

---

## 18. Suggested Screenshot Order for Lab Record

Keep the screenshots in this order so the experiment is easy to check:

1. Final topology
2. R1 `show ip interface brief`
3. R2 `show ip interface brief`
4. R3 `show ip interface brief`
5. PC1 IP configuration
6. PC2 IP configuration
7. R1/R2 direct-link ping
8. R2/R3 direct-link ping
9. RIP configuration
10. R1 routing table
11. R2 routing table
12. R3 routing table
13. PC1 → PC2 successful ping
14. Wireshark ping capture
15. Detailed packet view

---

## 19. Result

A 2-hop network was designed in GNS3 using three routers, IPv4 addresses were configured on all required interfaces and PCs, RIP version 2 was configured for dynamic routing, the routing tables were verified, end-to-end connectivity was tested from PC1 to PC2, and packets were analyzed using Wireshark.


<br><br><br>

---

## 20. Source

Based on the **Computer Communication Networks Lab Manual Aug–Dec 2026, Experiment 6 & 7**, PES University, ECE. The manual covers the GNS3 topology, IPv4 interface configuration, RIP version 2, PC addressing, routing-table verification, ping testing, and Wireshark packet capture.
