*This project has been created as part of the 42 curriculum by jotriano.*

# NetPractice

## Description

NetPractice is a hands-on networking exercise from the 42 curriculum. Its goal is to discover the basics of computer networking by repairing small, **simulated** networks: configuring IP addresses and subnet masks, choosing default gateways, writing static routes, and understanding how hosts, switches and routers exchange packets.

The networks are not real. They are displayed in a local web interface (HTML and JavaScript) provided with the subject. Each of the 10 levels shows a broken network diagram and a list of goals, for example "host A needs to communicate with host B". Only the unshaded fields (IP addresses, masks, routes and gateways) can be edited. A goal is **OK** when the packet reaches its destination (*forward way*) **and** the reply comes back (*reverse way*). A level is solved when all of its goals are OK.

Overview of what the levels cover, from simple to complex:

- Two hosts on the same network: same subnet, same mask, valid host addresses.
- Several hosts connected through a switch.
- A router joining two or more networks, with hosts using it as their default gateway.
- Static routes (`destination => gateway`) on hosts and routers, including the return path.
- Subnetting with different mask sizes, and making sure that networks never overlap.
- Public versus private addresses, and a simulated Internet that does not route private ranges.

## Instructions

### Requirements

- A web browser.
- `python3` (used by `run.sh` to start a local web server).

### Running the training interface

1. Download the archive attached to the project page and extract it in any folder.
2. In that folder, run:

   ```sh
   ./run.sh
   ```

   The script starts a local web server and opens the browser on the interface. If it does not work, start the server manually and open the page yourself:

   ```sh
   python3 -m http.server 49242
   # then open http://localhost:49242 in the browser (the port can be changed)
   ```

3. In the **Training** tab, enter your intranet login (`jotriano`, exactly as written, lowercase) and press **Start!**. The login is used as a seed to generate the addresses of every level, so the same login always produces the same networks. It is very important to enter it before starting.
4. Edit the unshaded fields and press **Check again**. The logs at the bottom of the page explain why a goal fails (for example "destination does not match any route" or "multiple interface match").
5. When all goals are OK, press **Get my config** to download the configuration of the level, then press **Next level**.

The **Evaluation** tab generates random configurations (without a login), like the ones used during the defense.

### Exporting configurations

**Get my config** downloads one file per level, named `level1.json`, `level2.json`, ... `level10.json`. It must be pressed on every level, after the level is solved and before moving to the next one. The file stores the values of the editable fields (IP addresses, masks, routes and gateways) at that moment.

### Submission requirements

- The **10 exported configuration files** (`level1.json` to `level10.json`, one per level) are placed at the **root of the repository**, next to this `README.md`.
- Only the content of the repository is evaluated. File names are checked carefully.
- During the defense, three random levels must be solved in a limited time, without external tools (a simple calculator such as `bc` is tolerated).

```
.
├── README.md
├── level1.json
├── level2.json
├── ...
└── level10.json
```

## Method

The method used to solve every level:

1. **List the cables.** Each cable (or switch) is one network. All the interfaces on it must share the same network and mask, and every network must be different and must not overlap with another.
2. **Start from what cannot be edited.** Fixed addresses, fixed masks and fixed gateways give the constraints that the rest of the configuration has to follow.
3. **Compute the address block.** Block size = 256 minus the mask octet where the mask is cut. Blocks start at multiples of the block size. The first address of a block is the network address, the last one is the broadcast address, and the addresses between them are usable by hosts.
4. **Set gateways and routes.** A host's gateway is the IP of the router interface on its own network. A router directly knows the networks of its interfaces and needs a route for any other network. Routes are written with the exact network address, for example `147.131.41.192/27`.
5. **Check both directions.** For each goal, trace the forward way and the reply.

Some pitfalls found in the simulator:

- A mask that is too large makes a router believe that a destination is directly connected and skips its routing table (interfaces are checked before routes).
- Two networks that overlap make the router unable to choose an interface (`multiple interface match`).
- Only the first matching route is used, so more specific routes must come before a default route.
- The Internet does not route private ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), so hosts that must talk to the Internet need public addresses.

## Resources

### Networking concepts studied

- **TCP/IP addressing**: IPv4 addresses as four octets, binary and decimal conversion, network, host and broadcast addresses.
- **Subnet masks and CIDR notation**: mask octets, prefix length (`/24`, `/27`, ...), block sizes, subnetting.
- **Default gateways and static routes**: next hop, default route (`0.0.0.0/0`), route matching, forward and return paths.
- **Routers and switches**: a switch connects devices of the same network, a router connects different networks and forwards packets between them.
- **OSI layers**: the project works mainly on layer 3 (network: IP addressing and routing); switches operate on layer 2 (data link).
- **Private and public addresses**: ranges reserved by RFC 1918 and why they are not routed over the Internet.

### References

- RFC 791, *Internet Protocol*: https://www.rfc-editor.org/rfc/rfc791
- RFC 1918, *Address Allocation for Private Internets*: https://www.rfc-editor.org/rfc/rfc1918
- RFC 4632, *Classless Inter-domain Routing (CIDR)*: https://www.rfc-editor.org/rfc/rfc4632
- IPv4: https://en.wikipedia.org/wiki/IPv4
- Subnetwork: https://en.wikipedia.org/wiki/Subnet
- Classless Inter-Domain Routing: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
- Default route: https://en.wikipedia.org/wiki/Default_route
- Router (computing): https://en.wikipedia.org/wiki/Router_(computing)
- Network switch: https://en.wikipedia.org/wiki/Network_switch
- OSI model: https://en.wikipedia.org/wiki/OSI_model

### Use of AI

I used an AI assistant (Claude, by Anthropic) as a tutor and reviewer, not as a replacement for my own work:

- **Understanding the subject**: translating and explaining the English subject and the vocabulary of the project.
- **Learning the concepts**: explaining binary and decimal conversion, subnet masks, block sizes, broadcast and network addresses, gateways and static routes, with examples.
- **Guiding the levels**: explaining the reasoning step by step for each level (what is fixed, which network each cable forms, which route is needed and why), and helping me read the simulator logs when a goal failed, for example overlapping networks or a mask that was too large.
- **Writing**: helping to draft and structure this README.

Every configuration was entered and tested by me in the training interface with the **Check again** button, and I went through the reasoning of each level so that I can explain it during the evaluation.
