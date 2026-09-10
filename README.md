# Network-Projects
# 30 Days, 30 Networking Projects 🚀

Hey there! 👋 I'm Abdelrahman, an IT student with my eyes set on the Blue Team.

I figured out pretty quickly that to defend a network, you first need to understand how it actually *ticks* — and get really comfortable living in Linux. So I set a challenge for myself: **30 days, 30 networking projects.**

In this repo, I'm sharing the journey raw and real — every project, what I actually learned, and the roadblocks and bugs that drove me crazy along the way.

---

## 🗓️ Day 1 — First LAN, First Subnet Mask Mystery

**The Setup:**
- 2 PCs
- 1 Switch

**Experience:**

I connected 2 PCs through a switch and set up the IPs manually. PC0 got `192.168.1.10 / 255.255.255.0`, PC1 got `192.168.1.11 / 255.255.255.0`. Pinged PC1 from PC0's command prompt — worked perfectly.

Then I got curious and changed PC1's subnet mask to `255.255.255.128`, just to see what would happen. The ping still worked, no difference at all — which confused me at first, until I broke down *why*: `255.255.255.0` covers one giant block (`0–255`), while `255.255.255.128` splits that same range into two blocks (`0–127` and `128–255`). Both PCs' IPs (`.10` and `.11`) happened to land in the *same* block (`0–127`), so as far as the network was concerned, nothing actually changed.

To actually see it break, I changed PC1's IP to `192.168.1.130` instead. That address falls in the *second* block (`128–255`) — a completely different neighborhood than PC0's `.10`. Ping failed immediately, exactly as expected once I understood the block logic.

**What I Learned:**
1. What a subnet mask actually is, and what it really does.
2. What happens when you change a device's subnet mask but keep its IP the same.
3. What happens when you also change the IP itself.
4. The block/range rule — the mask decides how a whole address range gets split into "neighborhoods," and two devices can only talk directly if they land in the *same* one.

**Challenges:**

The block/range rule was genuinely confusing at first — my brain kept trying to treat it as one flat rule instead of something built from binary underneath. I broke it down piece by piece (binary switches → locked vs free bits → block size → block boundaries) until it actually clicked, instead of just memorizing "128 = 2 blocks."

**Conclusion:**

Such a fun first experience, and it's already reshaping how I look at IP addresses — not just as random numbers, but as something with real structure underneath. Excited for what's next. 🙌

---

## 🗓️ Day 2 — Scaling Up: 1 Router, 4 Subnets, 8 PCs

**The Setup:**
- 1 Router
- 4 Switches
- 8 PCs

**Experience:**

Today I built a bigger map — my first time actually using a router. I took Day 1's basic setup, duplicated the switch+PC pattern 3 more times, and connected everything through one router (with extra interface modules added to get 4 usable ports).

The first real challenge was assigning IPs. I used:
- PC0 & PC1: `192.168.10.10` / `.11`
- PC2 & PC3: `192.168.10.70` / `.71`
- PC4 & PC5: `192.168.10.130` / `.131`
- PC6 & PC7: `192.168.10.205` / `.206`

All 8 PCs shared the same subnet mask: `255.255.255.192` (/26).

Coming out of Day 1, I was still a little fuzzy on how devices with *different* IPs could all be organized under one mask and still make sense as separate networks. Day 2 is where it actually clicked: `255.255.255.192` splits the range into **4 blocks** — `0–63`, `64–127`, `128–191`, `192–255` — and I deliberately picked IPs so each pair of PCs landed in a *different* block from the others. That means, right now, none of them see each other as "local" — each PC assumes the others live on a different network entirely, so direct communication isn't possible yet.

**What I Learned:**
1. How to take one big network (`/24`) and mathematically split it into smaller, equal subnets by borrowing bits.
2. How to correctly identify a subnet's network address, broadcast address, and usable host range — and why the first and last address in every block are off-limits for devices.
3. That a subnet mask alone doesn't guarantee two devices can talk — the actual IP values have to land in the same block too.
4. That even if two devices logically "agree" they're in the same block, physical topology still matters: separate switches need a router to actually forward traffic between them — a shared mask isn't enough by itself.
5. Real router config basics: adding interface modules, assigning IPs to interfaces, and the classic first-timer mistake of forgetting `no shutdown` (spent a while confused by red link triangles before realizing the interfaces were administratively down).

**Challenges:**

Figuring out *why* my carefully-chosen IPs still couldn't ping each other took some back-and-forth — I initially thought it was purely a subnet-mask issue, but the real answer combined two things: different IP blocks **and** no routing configured yet between the router's interfaces. Also spent a chunk of time debugging red "not connected" triangles on the router links, which turned out to be shut-down interfaces, not a cabling problem.

**Conclusion:**

This project took things from "two PCs talking" to an actual small multi-network topology — and for the first time, I really felt the difference between *addressing* (which subnet is this?) and *routing* (can traffic actually get there?). Day 3 is going to dig into what's actually happening inside those packets with Wireshark — can't wait. ⚡

## 🗓️ Day 3 — OSI Model, For Real (Wireshark)

**The Setup:**
- Wireshark
- A browser + neverssl.com (plain HTTP, no encryption, perfect for reading raw traffic)

**Experience:**

Today was my first time ever opening Wireshark, and not gonna lie, seeing hundreds of rows scroll by the second I started capturing was a little overwhelming at first. But once I picked my active network interface, started the capture, and visited `neverssl.com`, things got a lot more manageable — I filtered the traffic down using `http` in the display filter bar, and suddenly instead of chaos I had just a handful of relevant rows.

I found a packet with `GET /online HTTP/1.1` in it and clicked into it. This is where it got really cool — instead of reading about the OSI model as a 7-layer diagram in a slide, I was staring at the actual layers stacked inside one real packet:

- **Ethernet II** → Source MAC `38:54:9b:30:e5:44` (my laptop), Destination MAC `58:00:e3:f0:bf:2b` (my router — NOT the actual website, which threw me for a second until I understood why).
- **Internet Protocol** → Source `192.168.1.5` (my private IP), Destination `34.223.124.45` (neverssl.com's real public server IP).
- **TCP** → Source port `51811` (random, picked by my OS for this connection), Destination port `80` (standard HTTP), Flags `PSH, ACK` (mid-conversation, pushing data + acknowledging).
- **HTTP** → `GET /online HTTP/1.1` with `Host: neverssl.com` — the actual human-readable request.

The biggest "aha" moment was realizing MAC addresses only ever describe the **next hop** on the local network, not the true final destination — my PC doesn't know or care about neverssl.com's MAC address, it just needs to hand the packet to my router, which figures out the rest. IP, on the other hand, is the layer that actually knows the real end-to-end destination across the internet.

**What I Learned:**
1. How to actually use Wireshark — pick an interface, capture, filter, and read a packet layer by layer.
2. The real difference between MAC addresses (physical, fixed, local-hop-only) and IP addresses (logical, assigned, true end-to-end addressing) — MAC won't change if I switch networks, but IP will, since it depends on whichever network I'm connected to.
3. What TCP ports and flags actually represent in a live connection — a random source port per connection, a fixed destination port for the service, and flags describing the conversation's current state.
4. The full 7-layer OSI model (Physical, Data Link, Network, Transport, Session, Presentation, Application) — but more importantly, that in real-world traffic you mostly see 4 concrete layers clearly (Ethernet, IP, TCP, and the application protocol like HTTP), while Physical/Session/Presentation stay mostly conceptual in day-to-day packet analysis.

**Challenges:**

Wireshark's sheer amount of live traffic was intimidating for about the first two minutes — my instinct was to panic a little at all the noise. Filtering by `http` fixed that immediately, and once I saw the packet broken into those 4-5 expandable layers, it stopped feeling like chaos and started feeling like a structure I could actually read.

**Conclusion:**

This was the day OSI stopped being a diagram I memorized for an exam and became something I could point at inside real traffic. Feels like a genuine shift from "studying networking" to "reading networking" — excited to bring routing into the picture next and finally get those 4 subnets from Day 2 talking to each other. ⚡

## 🗓️ Day 4 — Static Routing (and a LOT of troubleshooting)

**The Setup:**
- 2 Routers (R1, R2)
- 2 Switches
- 4 PCs
- 1 Serial link between the routers

**Experience:**

Today was supposed to be about learning `ip route`, but it turned into the most hands-on troubleshooting day so far — which honestly ended up being the real lesson.

I started by extending my Day 2 topology with a second router, but hit a wall almost immediately: a module (`NM-1CFE`) on the router refused to accept a cable connection no matter what I tried. After going back and forth trying to diagnose it (wrong cable type, wrong module, router power state), I made the call to stop fighting the broken build and start fresh instead — 2 clean routers, no extra modules, connected directly via a serial link this time.

Once rebuilt, I configured both routers' interfaces (`ip address` + `no shutdown` on each), then added static PCs on both sides — subnet `192.168.10.0/24` on R1's side, `192.168.20.0/24` on R2's side.

First real test: pinging across from R1's subnet to R2's subnet **before** adding any routes. It failed with `Destination host unreachable` — and that specific error (not just a timeout) told me something important: R1 actually received the packet and tried to route it, but had no idea where `192.168.20.0/24` was, so it rejected it outright.

Then I added the actual static routes:
ip route 192.168.20.0 255.255.255.0 10.0.0.2 (on R1)
ip route 192.168.10.0 255.255.255.0 10.0.0.1 (on R2)

Ran the ping again — 3 out of 4 replies came back successfully (the first one timed out, which turned out to be normal ARP resolution delay on the very first packet, not an actual problem).

**What I Learned:**
1. A single router automatically routes between ALL of its own directly-connected subnets — no static routes needed for that case. Static routes are only required when a network sits *beyond* another router.
2. The exact syntax and logic of `ip route [destination network] [mask] [next-hop]` — "to reach this network, send traffic to this next device."
3. The real difference between `Destination host unreachable` (router actively rejected it — no matching route) vs a plain timeout (no response at all) — I'd only read about this distinction before, today I actually generated and diagnosed it myself.
4. How to troubleshoot a broken physical connection systematically — checking module types, checking `show ip interface brief`, trying different cable-connection modes, and knowing when to rebuild clean instead of endlessly debugging a broken setup.
5. Serial link basics — connecting two routers directly with a serial interface instead of going through a switch.

**Challenges:**

This was by far the most frustrating day yet. A router module (`NM-1CFE`) flatly refused a cable connection, and I spent a long time trying different cable types and module swaps before deciding to rebuild the whole topology from scratch with simpler routers instead. It was a little discouraging in the moment — this day didn't have the same "aha!" feeling Day 1 and 2 did, since the core routing *concept* was actually simple; the difficulty was almost entirely tooling friction, not understanding. Had to remind myself that fighting through broken configs and misbehaving hardware is a real, valid part of this skill set too — not just theory clicking into place.

**Conclusion:**

Less of a "lightbulb" day and more of a "grind it out" day — but I think those matter just as much, if not more, for actually being job-ready. I now have real reps at diagnosing why a connection isn't working instead of just knowing the commands in isolation. Planning to pause after Day 5 and run myself through a proper test covering everything from Days 1–5 before pushing further, to make sure it's all actually sticking and not just day-by-day survival. Onward to DHCP next. ⚡
