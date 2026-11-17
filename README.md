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

    
## Week 6: tunnels, MTU and containers

- **Published page:** https://miraqilah.github.io/netlab/week6/
- **What I did and why:** 
    - **Lab, Linux building blocks:** I explored how Mininet hosts and links are created using Linux network namespaces, virtual Ethernet pairs and bridges. This helped me understand how Mininet provides network isolation and connects hosts through virtual interfaces. 
    
    - **Lab, Building a VXLAN tunnel:** I created a VXLAN tunnel between h1 and h2 using VXLAN ID 100 and UDP port 4789. I assigned IP addresses to both VXLAN interfaces and tested their connectivity. I also used tcpdump to observe how packets are encapsulated. This helped me understand how VXLAN tunnel is invisible to switch. Part 3 showed s1 forwarding the packets “without knowing there was a tunnel at all”. That is why VXLAN works over almost any network, and why upgrading is much easier. While in P4, the switches were the tunnel. When s2 ran the old program in the mixed rollout, it misread the header (Bytes parsed: 38). Every device on the path must agree.

    - **Lab, MTU and Throughput:** I checked the MTU of the underlay and VXLAN interfaces and compared their network performance using iperf3. The underlay MTU was 1500 bytes, while the VXLAN MTU was 1450 bytes due to the 50-byte IPv4 encapsulation overhead.

    - **Lab, VXLAN over IPv6:**  I configured IPv6 addresses on the underlay interfaces and created another VXLAN106 tunnel using IPv6 as the outer transport. The encapsulation overhead increased from 50 to 70 bytes because the IPv6 header is 20 bytes larger than the IPv4 header.
    
    - **Lab, MTU Mismatch:** I changed h2's underlay interface MTU from 1500 to 1400 bytes while leaving the other side unchanged. Small pings still worked, but larger packets were dropped. This is because small ping packets sizes were below the available MTU limit. Therefore, they could travel through the network without being dropped. I also checked packet-drop counters and tested TCP performance to understand how MTU mismatch can cause packet loss and retransmissions. TCP transfers larger amounts of data and may generate packets that exceed the MTU limit. When packets become too large, they may be dropped, causing TCP retransmissions and reducing the overall throughput. This explains why small pings could still succeed while TCP performance was affected by the MTU mismatch.

    - **Lab, VXLAN MTU limitation:** I tested different MTU settings and observed how Linux rejects an invalid VXLAN MTU configuration when it exceeds the available underlay MTU.
    
    - **Lab, Docker Containers:** I created an Alpine Linux Docker container and explored its network interface, filesystem and running processes. This helped me understand how Docker containers differ from Mininet hosts, as containers provide additional isolation through network, process and mount namespaces.

    - **Assignment, design data centre:** I edited `overlay.yaml` to describe my own network with 3 hypervisors and 3 tenants, each with its own VNI. I also configured overlapping IP addresses between tenants to understand how separate virtual networks can use the same IP address without interfering with each other. Two tenants span 2 hypervisors each, while one tenant spans all 3 hypervisors. Each hypervisor has one virtual machine from each tenant.

    - **Assignment, build the tunnel endpoints:** fill up `tenant_commands()` in `build_overlay.py` to create the tenant network interfaces and VXLAN tunnel endpoints. This allows VMs belonging to the same tenant to communicate even when they are located on different hypervisors.

    - **Assignment, Check the MTU:** implemented `check_mtu()` to check whether the configured tenant MTU and VXLAN encapsulation overhead exceed the maximum underlay MTU. The function stops the network when it detects an MTU problem instead of allowing a network with an invalid MTU configuration to run. 

    - **Assignment, Evidence from the running overlay:** 
        - **Flood:** I wanted to understand how VXLAN handles flooding and MAC learning. First, I checked the FDB on hv1 before sending any traffic. I found that it already had permanent flood entries pointing to hv2 and hv3. Then, I sent a ping from red1 on hv1 to red2 on hv2. At the same time, I used tcpdump on hv3 to capture VXLAN traffic. Although hv3 was not the destination of the ping, I managed to capture a packet from hv1 to hv3 using UDP port 4789 and VNI 700. This showed that unknown traffic was flooded to other remote VTEPs, not only the intended destination. Finally, I checked the FDB again and observed that the hypervisors had learned the remote MAC addresses. This demonstrates how MAC learning helps the network forward subsequent traffic more efficiently."

        - **Isolation:** I tested whether two different tenants could use the same IP address without interfering with each other. First, I pinged 10.80.50.11 from red1 and checked its neighbour table. The MAC address returned was 02:00:02:bc:00:0b, which belongs to red2. At the same time, I captured packets on blue2, but no packets were captured. This shows that traffic from tenant red was not reaching blue2. Next, I pinged the same IP address from blue1. The neighbour table showed a different MAC address, 02:00:03:20:00:0b, which belongs to blue2. Although both tenants used the same IP address, each resolved it to its own VM. This demonstrates how VXLAN uses different VNIs to maintain tenant isolation and allow overlapping IP addresses in separate virtual networks.

        - **Black hole:** Performed a normal ping between red1 and 10.80.50.11 to confirm that basic network connectivity was working. Then, tested with a larger packet size of 1472 bytes. The ping failed because the packet exceeded the available MTU after VXLAN encapsulation, causing it to be dropped. Next, used iperf3 to test the impact of the MTU issue on TCP traffic. Started an iperf3 server on red2 and connected to it from red1. Although the TCP connection could be established, the data transfer experienced problems because oversized packets were dropped along the path. Finally, performed DF (Don't Fragment) ping tests with different packet sizes to identify the MTU boundary. Sent a packet with a size of 1450 bytes (1422 bytes ICMP payload + 28 bytes headers). The packet was successfully delivered with 0% packet loss. Increased the packet size to 1451 bytes (1423 bytes ICMP payload + 28 bytes headers). The packet failed to reach the destination. This shows that the path supports an MTU of 1450 bytes. Packets larger than this cannot be delivered without fragmentation.

        - **Fixes:** Tested two different solutions to understand how adjusting the tenant MTU and increasing the underlay MTU can prevent oversized packets from being dropped in the VXLAN overlay. 
            - **Fix A: Tenant MTU 1450:** Tested packet sizes of 1450 bytes (ping success) and 1451 bytes (ping failed) using DF ping. Used iperf3 to check TCP data transfer after applying the tenant MTU adjustment.

            - **Fix B: Underlay MTU 9000:** Tested packet sizes of 1500 bytes (ping success) and 1501 bytes (ping failed) using DF ping. Then repeated the iperf3 test to check TCP data transfer with the larger underlay MTU.

    - **Page:** `make_page6.py` to generate the Week 6 page using the network configuration and page data. This helps keep the published page consistent with the network design and its checks.


- **Files, lab:**
    - `week6/Part1-7.txt` – Contains the commands I ran and their terminal outputs for all seven parts.

- **Files, assignment:**
    - `overlay.yaml` : data centre network design 
    - `build_overlay.py`    : to build the data centre
    - `blackhole.yaml` : all the tenant's and underlay MTU is 1500
    - `mtu_a.yaml` : every tenant's MTU lowered to 1450; the underlay stays at 1500
    - `mtu_b.yaml` : tenants MTU stay at 1500; the underlay's MTU is raised to 9000
    - `evidence6/check.txt`: `check_mtu()` stopping `blackhole.yaml` with PROBLEM line per mistake
    - `evidence6/flood.txt` : how VXLAN using flood to reach the unknown traffic; MAC address leaning
    - `evidence6/isolation.txt` : Evidence showing same IP address used on seperate tenant will not interfer
    - `evidence6/blackhole.txt` : Ping and iperf3 results demonstrating the MTU blackhole problem.
    - `evidence6/fixes.txt`     : Ping and iperf3 measurements from the two MTU fixes
    - `evidence6/mtu_a_check.txt` : Evidence of the MTU verification for Fix A
    - `evidence6/mtu_b_check.txt` : Evidence of the MTU verification for Fix B
    - `make_page6.py` : Generates the published page.
    - `page6.yaml` : Answer for lab 6 assignment


- **Run it:**

      mn --topo single,2 --mac --switch lxbr --controller none --ipbase 10.NN.0.0/24   # builds the lab network that has a switch with 2 hosts connected to it
      python3 build_overlay.py overlay.yaml             # builds the data centre
      python3 build_overlay.py blackhole.yaml           # network with black-hole design that tenant's MTU > underlay MTU (1500 bytes)
      python3 build_overlay.py mtu_a.yaml               # `check_mtu()` to run with MTU setting inside mtu_a.yaml
      python3 build_overlay.py mtu_b.yaml               # `check_mtu()` to run with MTU setting inside mtu_b.yaml
      python3 build_overlay.py blackhole.yaml --force   # build black-hole network after `check_mtu()` is filled
              

- **AI use**
    - **Tool:** ChatGPT (OpenAI), through the ChatGPT web application.

    - **Lab** ChatGPT helped me with:
        - Explaining Linux network namespaces, virtual Ethernet interfaces, Linux bridges and how Mininet hosts are connected.
        - Explaining how VXLAN encapsulation works and how to configure a VXLAN tunnel between two Mininet hosts.
        - Understanding the packet headers and the difference between IPv4 and IPv6 VXLAN overhead.
        - Explaining MTU, packet fragmentation, dropped packets, TCP retransmissions and throughput measurements.
        - Troubleshooting commands and understanding the output from ping, tcpdump, iperf3 and Docker.
        - Explaining the differences between a Mininet host and a Docker container.


    - **Assignment** ChatGPT helped me with:
        - Understanding the purpose of the network configuration files and the functions in build_overlay.py.
        - Troubleshooting the VXLAN overlay, tenant isolation and MTU blackhole experiments.
        - Understanding the results from the flood, isolation, blackhole and MTU fix experiments.
        - Improving the explanations and organisation of this README.

    - I ran all the commands myself in the lab environment and checked the results against the terminal outputs. I also tested the tunnel, changed MTU values and observed how the network behaved under different conditions. I take responsibility for the work I submitted, and I am ready to explain any part of it.