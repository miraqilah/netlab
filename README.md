# netlab: network automation labs

Lab portfolio for WIC2005. Everything here runs on my own addresses
(last 4 digits of my student ID: 1780, so IPv4 10.80.0.0/16 and IPv6 fd00:1780::/32).

## How to run anything here

You need Git and Docker Desktop.

    git clone https://github.com/miraqilah/netlab
    cd netlab
    docker run -it --rm --privileged -v "${PWD}:/lab" -w /lab firdaussahran/netlab-mininet:1.0

Commands below are typed inside the container, from /lab.
To use your own addresses, replace 1780 and 80 with your own digits.

## Week 2: a network of my own design

- **Published page:** https://miraqilah.github.io/netlab/
- **What I did and why:** 
    - **Lab:** `build_topo.py` and `topology.yaml` to describe a network as data (one switch, three hosts) and let a script build it in Mininet. I made the file mine by replacing the placeholders with my own address blocks. I grew the network by editing the data (a second switch and a fourth host) and saw `git diff` show exactly what changed. I converted the YAML to JSON and built from both: the network is identical, but the JSON lost the comments. This shows that comments belong to the file format (the serialisation), not to the data model. I then gave h4 the same addresses as h2. The network still built, `pingall` reported only a small failure and never said why, and IPv6 noticed the duplicate (`dadfailed`) while IPv4 accepted it silently. Neither stopped the bad network from being built, so I filled in `check()` in `build_topo.py` to report any IPv4 or IPv6 address used by more than one host.

    - **Assignment, the design:** Edit `topology.yaml` to describes my own network with 3 switches and 6 hosts, each with an IPv4 and an IPv6 address from my own blocks. The switches form a line, s1 – s2 – s3: s1 has two hosts (h1 and h2), s2 has one (h3) and s3 has three (h4, h5 and h6). I chose the layout freely, and a line is the simplest way to join the switches.  

    - **Assignment, the checks:** I made `check()` stricter so that it also reports any link that names a switch or host that doesn't exist, because a file can be valid YAML and still describe a wrong network. I made `bad.yaml`, a deliberately broken copy with a duplicate address and a link to a switch that doesn't exist, to prove that `check()` catches both mistakes.

    - **Assignment, the evidence:** I collected evidence from the running network (listed below), which shows IPv4 and IPv6 answering the same question ("which MAC has this address?") in different ways, what the switch learned by itself, and IPv6 rejecting a duplicate address.

    - **Page:** `make_page.py` generates the page from the same data, so the page and the network can't disagree. It refuses to publish a network that fails a check.

    
- **Files, lab:**
    - `topology.yaml`: the network, as data (the lab started with one switch and three hosts, and I later edited it into my own design)
    - `build_topo.py`: checks the description, then builds the network in Mininet
    - `topology.json`: the same network in JSON format, which lost the comments

- **Files, assignment:**
    - `topology.yaml` and `build_topo.py`: my 3-switch, 6-host network, and the checker with my `check()`
    - `bad.yaml`: the deliberately broken copy (h1 and h5 has same address, there is link between h1, s9 which s9 is not exist)
    - `make_page.py` and `page.yaml`: generate the page, and hold my answers to the questions
    - `docs/index.html`: the generated page
    - `evidence/check.txt`: `check()` stopping `bad.yaml` with one PROBLEM line per mistake
    - `evidence/pingall.txt`: show every host can reach every other host
    - `evidence/neigh.txt`: h1's neighbour table, with IPv4 learned by ARP and IPv6 by Neighbour Discovery
    - `evidence/macs.txt`: which MAC address switch s1 learned behind which port
    - `evidence/dad.txt`: IPv6 marks the duplicate address `dadfailed`, while IPv4 accepts it without a word

- **Run it:**

      python3 build_topo.py topology.yaml   #builds the network in Mininet 
      python3 build_topo.py bad.yaml        #shows the PROBLEM lines that check() produces
      python3 make_page.py topology.yaml    #to generate the page to docs/index.html

- **AI use**
    - **Tool:** Claude (Anthropic), through the claude.ai chat. 

    - **Lab** Claude helped me with:
        - explaining concepts: data model vs serialisation, ARP vs IPv6 Neighbour Discovery, Duplicate Address Detection, MAC learning, and what the Git commands do
        - explaining error messages, for example the GitHub login failure that needed a personal access token
        - suggesting the first draft of `check()` in `build_topo.py` (the duplicate IPv4 and IPv6 address check). I put it in my file, ran it on duplicate data, and checked that its output matched what the lab expected.

    - **Assignment** Claude helped me with:
        - suggesting the extra check in `check()` that reports a link naming a switch or host that doesn't exist
        - suggesting the code for `links_section()` and `mermaid_diagram()` in `make_page.py`
        - explaining what each question in `page.yaml` was asking and giving feedback on my draft answers. 
        - guiding the GitHub Pages setup, and giving feedback on the wording of this README

    - I ran all the code and commands myself in the course container, and I checked the results against my own outputs and corrected my own mistakes. I tested my checks on both the working and the broken networks to make sure they behave the way I expect. I take responsibility for the work I submitted, and I am ready to explain any part of it.

## Week 3: a routed network

- **Published page:** https://miraqilah.github.io/netlab/week3/
- **What I did and why:** 
    - **Lab:** Built the lab's three-router network with `routers.py` and acted as the control plane. I switched forwarding on, added overlapping static routes, and saw longest-prefix match prefer the more specific `/24` over the `/20`. I then deleted routes to see the fallback and the unreachable case, made a routing loop (the TTL is what stops it) and repeated the routes in IPv6. Finally I moved the routes into `routes.yaml` and filled in `check_paths()` in `routers.py`, so a loop or a missing route is caught before any router is configured.

    - **Assignment, the design:** `network.yaml` describes my own network with 4 routers and 4 LANs, one host each. r1, r2 and r3 form a triangle, which gives the redundant path, and r4 is a stub router connected only to r1. Every router has static routes. r4 has one summary route, `10.80.0.0/22` via r1, which covers LANs 1, 2 and 3 but not LAN 4, so one route replaces three.

    - **Assignment, the checks:** I filled in `check_paths()` in `build_routed.py` to follow a packet router by router for every pair of hosts and report a router with no route or a loop. I made `loop.yaml` (r2 and r3 send LAN 4's traffic to each other) and `missing.yaml` (r3's route to 10.80.2.0/24 deleted) to prove it. I did this because static routes are not checked by anything: a bad route only shows up when traffic fails.

    - **Assignment, the failure:** when I took the r1–r2 link down, r1 lost its only route to h2's LAN and the trace stopped at r1, even though h2 was still reachable through r3. Static routes do not adapt.

    - **Page:** `make_page3.py` generates the page from `network.yaml` and refuses to publish a network that fails a check.

    - Collected evidence from the running network that I design earlier (listed below) under `evidence3/` .


- **Files, lab:** 
    - `routers.py`  : build lab's 3-router network 
    - `routes.yaml` : the five static IPv4 routes 
    - `lab3-test-loop.yaml`: broken copy where r3's `/20` points back at r1 creating a loop
    - `lab3-test-missing.yaml`: broken copy with r2's route to h1's LAN deleted
    - `week3/r1-routes-before.txt`  : r1's table before any routes (connected networks only)
    - `week3/lpm.txt`   : which route r1 picks for h2, h3 and an unused address
    - `week3/trace-h2.txt`  : traceroute from h1 to h2
    - `week3/trace-h3.txt`  : traceroute from h1 to h3
    - `week3/trace-h2-fallback.txt` : traceroute from h1 to h2 after deleting r1's `/24`, h2 is reached via r3 using the `/20`
    - `week3/trace-h2-unreachable.txt`  : traceroute from h1 to h2 after after also deleting r3's `/20`, the trace stops at r3 with `!N`
    - `week3/trace-loop.txt`    : traceroute bouncing between h1 to h2 during loop 
    - `week3/lpm-ipv6.txt`  : which route r1 picks for h2, h3 in IPv6
    - `week3/trace-h3-ipv6.txt`: traceroute from h1 to h3 in IPv6

- **Files, assignment:**
    - `network.yaml`: my network design
    - `build_routed.py`: checks the network (including my `check_paths()`), then builds it
    - `loop.yaml` and `missing.yaml`: deliberately broken copies
    - `make_page3.py` and `page3.yaml`: generate the page, and hold my answers
    - `docs/week3/`: the generated page
    - `evidence3/check.txt`: `check_paths()` stopping the loop and the missing route
    - `evidence3/routes.txt`: every router's routing table
    - `evidence3/traces.txt`: traceroutes from h1 to h2, h3 and h4
    - `evidence3/summary.txt`: r4 sends h1, h2 and h3 to the same next hop (the `/22` at work)
    - `evidence3/failure.txt`: the trace from h1 to h2 stopped at r1 after the r1–r2 link went down


- **Run it:**

      python3 routers.py --id 1780                                  # lab network without routing table, host can't reach each other
      python3 routers.py --id 1780 --routes routes.yaml             # lab network with route
      python3 routers.py --id 1780 --routes lab3-test-loop.yaml     # lab network with the looping route
      python3 routers.py --id 1780 --routes lab3-test-missing.yaml  # lab network with the missing route 
      python3 build_routed.py network.yaml                          # running the design network
      python3 build_routed.py loop.yaml                             # stops with PROBLEM lines, nothing built
      python3 build_routed.py missing.yaml                          # stops with PROBLEM lines, nothing built
      python3 make_page3.py network.yaml                            # generate page

- **AI use**
    - **Tool:** Claude (Anthropic), through the claude.ai chat. 

    - **Lab** Claude helped me with:
        - explaining commands, forwarding (`ip_forward`), static routes, longest-prefix match (including the binary view of `/24` vs `/20`), TTL and routing loops
        - explaining error messages and typing mistakes, for example `RTNETLINK answers: No such process` and a garbled command
        - suggesting the first draft of `check_paths()` in `routers.py`. I tested it on `lab3-test-loop.yaml` and `lab3-test-missing.yaml` and checked that it reported the loop and the missing route.

    - **Assignment** Claude helped me with:
        - checking my network design against the requirements. I came up with the topology myself (4 routers, 4 LANs, a triangle r1–r2–r3 with r4 as a stub on r1, and my own address numbering) and also the `routes:` section of `network.yaml`, including the `10.80.0.0/22` summary route on r4. 
        - suggesting the code for `check_paths()` in `build_routed.py` and `routing_tables_section()` in `make_page3.py`
        - explaining how to make the two broken copies (`loop.yaml`, `missing.yaml`) and which commands to use for the evidence
        - explaining the `page3.yaml` questions and giving feedback on my draft answers. 
        - helping with the README structure and with how to test that my work is reproducible
    
    - I ran all the code and commands myself in the course container, and I checked the results against my own outputs and corrected my own mistakes. I tested my checks on both the working and the broken networks to make sure they behave the way I expect. I take responsibility for the work I submitted, and I am ready to explain any part of it.


## Week 5: a hop limit for the P4 tunnel

- **Pull request:** https://github.com/miraqilah/p4-lab/pull/1
- **What I did and why:**
    - **Lab:** I ran P4 program (`tunnel.p4`) on two software switches in Mininet. I watched it forward IPv4 and read the switch logs to see the parser states, the table key, hit or miss, and the action. I changed a table while the switch was running (`table_delete` and `table_add` with `p4cli`), which showed that table entries are run time and a new header needs a new program (compile time). I then finished the custom tunnel header: the parser reads it (`extract`), the deparser writes it back (`emit`), and the ingress uses `myTunnel_exact` to forward. In Step 6 I broke it on purpose by removing the `emit`. The compiler did not complain, but s2 read the IPv4 bytes as a tunnel header (`key is 4500`, `dst_id` 0022), found no exit 34 and dropped the packet. This shows that the parser, the ingress and the deparser must agree, and that the compiler checks the code is valid, not that it is right.

    - **Assignment, analysis (tunnel loop):** I re-pointed tunnel exit 3 on s2 back to port 1, so s1 and s2 passed one packet back and forth. The pass counts kept rising after I sent only one packet (1422 to 1441 on s1, 1421 to 1441 on s2). From reading the code, the tunnel path (`myTunnel_forward`) never touches the TTL, so nothing counts hops. The IPv4 path (`ipv4_forward`) subtracts 1 from the TTL but nothing checks for 0, and because the TTL is 8 bits, 0 - 1 wraps to 255. So IPv4's TTL does not protect this program either.

    - **Assignment, design hop limit:** I added a 16-bit `hop_limit` field at the end of the tunnel header, after `dst_id`. I chose 16 bits so that all fields have the same size (the trade-off is 1 extra byte compared with 8 bits). I put it at the end so that the old fields stay in the same place. The sender starts it at 10 (the minimum is 2, because a packet crosses 2 switches). In the ingress, tunnelled packets check `hop_limit > 0`, subtract 1 and then look up `myTunnel_exact`. Otherwise they are dropped. The check comes before the subtraction so the value can never wrap around like the TTL does. The parser and deparser needed no edit, because `extract` and `emit` work on the whole header definition. `send.py` and `receive.py` had to change, because the header grew from 4 to 6 bytes and the hosts must agree with the switches.

    - **Assignment, the evidence:** With the new program, tunnelled packets reached h2 and h3 with `hop_limit=8` (10 minus 1 at each of the 2 switches), and a plain ping to h3 still worked (`ttl=62`). The same loop as before now ended after 11 passes (6 on s1 and 5 on s2): 10 forwards and 1 arrival that was dropped at 0. This matches my design.

    - **Assignment, the evaluation:** I ran s1 with my new program and s2 with the old one (`tunnel_v1.p4`). Below is what I was predicted:
        - s2 still read `dst_id` correctly and forwarded the packet out of port 2, but its log shows `Bytes parsed: 38` where the new program parses 40, so it started reading IPv4 2 bytes too early.
        - h2 printed nothing. I could not confirm why, because `tcpdump` captured 0 packets even for a working ping.
        - My prediction that the ping would not break because a plain IPv4 packet has no tunnel header, so both switches parse it the same way.
        - The loop ran 21 passes (11 on s1 and 10 on s2) instead of 11, because only s1 decrements the hop limit.

- **Files, lab:**
    - `tunnel.p4`: the P4 program 
    - `topo.py`: compiles the program, starts the two switches and Mininet, and fills the tables
    - `send.py` and `receive.py`: the host programs that send and print packets
    - `p4cli`: changes and shows the switch tables at run time
    - `lab-notes.md`: my answers to the wrap-up questions

- **Files, assignment:**
    - `tunnel.p4`, `send.py` and `receive.py`: the program with the hop limit (`--hops` sets the starting value, default 10)
    - `tunnel_v1.p4`: the finished lab program, kept unchanged as the "old" program for the mixed rollout
    - `assignment/REPORT.md`: my analysis, design, implementation, evidence and evaluation
    - `assignment/evidence/loop-before.txt`: the loop with the old program, pass counts growing
    - `assignment/evidence/delivery.txt`: tunnelled delivery to h2 and h3 with the hop limit, and a plain ping
    - `assignment/evidence/loop-after.txt`: the same loop with the hop limit, pass counts that stop
    - `assignment/evidence/mixed.txt`: s1 new and s2 old, including the failed `tcpdump` attempts

- **Run it:**

      python3 topo.py --id 1780                              # builds the network 
      python3 topo.py --id 1780 --s2-program tunnel_v1.p4    # mixed rollout: s1 new, s2 old

- **AI use**
    - **Tool:** Claude (Anthropic), through the claude.ai chat.

    - **Lab** Claude helped me with:
        - walking me through Steps 0 to 6 and explaining concepts: parser, ingress, deparser, tables, TTL, run time vs compile time
        - explaining the logs and output, for example hexadecimal addresses, table handles, `key is 4500` and `dst_id 0022`
        - checking my `lab-notes.md` answers and pointing out mistakes, such as mixing up `extract` and `emit`

    - **Assignment** Claude helped me with:
        - suggesting the hop-limit design options and the edits for `tunnel.p4`, `send.py` and `receive.py`, which I applied and tested myself
        - calculating why the loop gives 11 passes, and the idea for the mixed-rollout prediction
        - suggesting rollout options, data plane vs control plane points and cost points for the evaluation, and giving feedback on my report drafts

    - I chose the 16-bit size, the `> 0` check and the starting value of 10 myself (Claude suggested 8 bits and 8). I ran every command, produced all the evidence from my own network (`10.80.0.x`), wrote my prediction before the mixed run and did not change it, and corrected my own mistakes after checking them against the logs. I could not confirm what bytes reached h2, so that explanation is a hypothesis. I take responsibility for the work I submitted, and I am ready to explain any part of it.