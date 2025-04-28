# PCAP Analysis

This project aims to automate the process of identifying the most suitable deployment model for a workload when migrating solutions from an existing data center to a hybrid cloud. After capturing the network traffic we aim to extract logical identifiers that help to define the right deployment model for a workload in a hybrid cloud, by analyzing the PCAP file and understanding the network topology, communication patterns, and the identities of the communicating devices. We focus the analysis on:

### IP (Internet Protocol)

This is the fundamental Layer 3 protocol. IP addresses are the primary logical identifiers at Layer 3 and are present in virtually every packet captured in a PCAP file that involves IP communication. The IP header itself contains crucial logical identifiers:   

* **Source IP Address** identifies the sender of the packet.   
* **Destination IP Address** Identifies the intended recipient of the packet.   

### Routing Protocols (e.g., OSPF, BGP, RIP)

These protocols are used by routers to exchange routing information. Their packet payloads contain details about network topology, advertised routes, and neighbor relationships, which inherently include logical network identifiers (IP addresses of routers, network prefixes). Analyzing these packets can reveal the logical organization of the network.   

### Multicast Protocols (e.g., IGMP, PIM)

* **IGMP (Internet Group Management Protocol)** is used by hosts to join and leave multicast groups. IGMP messages contain the multicast group IP address (a logical identifier for a group of hosts). Examining IGMP reports and queries in a PCAP file shows which hosts are interested in which multicast groups.   
* **PIM (Protocol Independent Multicast)** os used by routers to manage multicast traffic. PIM packets contain information about multicast sources and groups, again involving logical IP addresses.   
* **Network Management Protocols (e.g., SNMP)** operate at the Application Layer (Layer 7), it often relies on UDP (Layer 4) and ultimately IP (Layer 3) for transport. The payload of SNMP packets (PDUs - Protocol Data Units) contains managed object information, which frequently includes logical identifiers like IP addresses, interface names, and system descriptions. Analyzing SNMP Get, Set, and Trap packets can reveal these identifiers associated with network devices.
* **Tunneling Protocols (e.g., GRE, IPsec)** encapsulate other packets within IP packets to create tunnels. The outer IP header contains the source and destination IP addresses of the tunnel endpoints (logical identifiers of the devices establishing the tunnel). Examining the encapsulated payload might reveal further logical identifiers from the original packets.   

## Extract identifiers from a PCAP File

Tools like Wireshark are invaluable for analyzing PCAP files. They provide powerful filtering and dissection capabilities to examine the headers and payloads of different Layer 3 protocols:   

* **Filtering by Protocol** allow to filter the capture to focus on specific Layer 3 protocols (e.g., ip, icmp, arp, ospf, bgp, igmp).
* **Examining Packet Details** that dissects the headers of each protocol, allowing you to easily view the source and destination IP addresses, protocol-specific fields, and payload data.
* **Following Streams** for connection-oriented protocols (though less common at pure Layer 3), you can follow streams to see the sequence of packets exchanged between specific logical endpoints.
* **Using Display Filters** create more complex filters to look for packets containing specific IP addresses or within certain IP ranges.   

Analyzing a PCAP file and focusing on Layer 3 protocols, cloud engineers extract a wealth of logical identifiers that help understand the network topology, communication patterns, and the identities of the communicating devices. However, these are logical identifiers; mapping them to physical machines often requires additional context or correlation with other information like ARP tables or network inventory data.

## Environment
Analyse PCAP files to identify deployment artifact types in a data center network. Main programming language is Python, virtual environments rely on [automatic shell activation](https://devenv.sh/automatic-shell-activation/) using devenv.sh and direnv.

## Initial Setup

It is recommended to prepare a python repository on github and clone the rope into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Add the python environment parameter to `devenv.nix`

```nix
  # Python environment
  languages.python = {
    enable = true;
    # https://devenv.sh/languages/python/
    # packages = [ pkgs.python39Packages.pip ];
    venv.enable = true;
    venv.requirements = ''
      requests
      # torch
    '';
    # venv.packages = [ pkgs.python39Packages.pip ];
    # venv.packages = [ pkgs.python39Packages.pip ];
    uv.enable = true;
  };
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
```

We add the dotfiles for the virtual environment to `.gitignore`

```sh
echo ".envrc" >> .gitignore
```

## Start analyzing

Before loading data, we initilaise the virtual environment

```sh
uv init && uv venv
source .devenv/state/venv/bin/activate
```



## Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
