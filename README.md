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
    - Building my own network that contains 3 switches and 6 hosts inside `topology.yaml` each with an IPv4 and an IPv6 address from my own address.

    - Converted the `topology.yaml` to `topology.json`. The network is identical, but the JSON lost the comments. This shows that comments are part of the file format, not of the data model.
   
    - In `build_topo.py`, filled in the check() function so it reports any address (IPv4 or IPv6) that used by more than one host, and also report any link that has switch or host that doesn't exist. The reason i did this because `pingall` only showed one failure out of 12 and gave no reason. IPv6 has dadfailed that notice the duplicate while IPv4 dont have anything like that. so yeah its better to stop bad network before it is built.

    - Created `bad.yaml` file to check the duplicate IP address and link to a switch that doesn't exist, this is to prove that `check()` catches both mistakes (`evidence/check.txt`).

    - Generated the web page from the same data with `make_page.py`, so the page and the network can't disagree.

    - Collected evidence from the running network (listed below).

- **Files:** 
    - `topology.yaml`   : the network 
    - `topology.json`   : network in JSON file
    - `build_topo.py`   : builds the network
    - `make_page.py`    : generates the page
    - `page.yaml`       : answer to the lab questions
    - `bad.yaml`        : deliberately broken copy of the network
    - `evidence/check.txt`  : check() on broken network
    - `evidence/pingall.txt`: show all the devices can reach each other
    - `evidence/neigh.txt`  : h1's neighbour table, IPv4 learn with ARP while IPv6 learn with neighbour discovery
    - `evidence/macs.txt`   : what switch s1's data plane learned by itself
    - `evidence/dad.txt`    : IPv6 marks the copy address as dadfailed; IPv4 accepts the duplicate without a word.
- **Run it:**

      python3 build_topo.py topology.yaml   #builds the network in Mininet 
      python3 make_page.py topology.yaml    #to generate the page to docs/index.html
      python3 build_topo.py bad.yaml        #shows the PROBLEM lines that check() produces

## Week 3: a routed network

- **Published page:** https://miraqilah.github.io/netlab/week3/
- **What I did and why:** [...]
- **Files:** `network.yaml`, `build_routed.py`, `make_page3.py`, `page3.yaml`, `loop.yaml`, `missing.yaml`, `evidence3/` (assignment), `routers.py`, `routes.yaml`, `week3/` (lab evidence)
- **Run it:**

      python3 build_routed.py network.yaml
      python3 make_page3.py network.yaml

## Week 4: OpenFlow (lab)

- **What I did and why:** [...]
- **Evidence:** `week4/`
- **Answers to the wrap-up questions:**
  1. [your answer]
  2. [your answer]
  3. [your answer]

## Weeks 5, 6, 7

[Add a section each week, as you finish it.]