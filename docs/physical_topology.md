# Feature Extraction

After capturing the network traffic we aim to extract logical identifiers that help to define the right deployment model for a workload in a hybrid cloud, by analyzing the PCAP file and understanding the network topology, communication patterns, and the identities of the communicating devices. We focus our analysis on the IP protocol, the fundamental Layer 3 protocol. IP addresses are the primary logical identifiers at Layer 3 and are present in virtually every packet captured in a PCAP file that involves IP communication. The IP header itself contains crucial logical identifiers:

* **Source IP Address** identifies the sender of the packet.  
* **Destination IP Address** Identifies the intended recipient of the packet.

Basic IP information is related to common network protocols, in order to understand the context for specific communication patterns.

## Extracting host identifiers

Network Tools like [Wireshark](https://www.wireshark.org/) and [scapy](https://scapy.net/) provide filtering and dissection capabilities to examine the headers and payloads of different Layer 3 protocols:

* **Filtering by Protocol** allow to filter the capture to focus on specific Layer 3 protocols (e.g., ip, icmp, arp, ospf, bgp, igmp).
* **Examining Packet Details** that dissects the headers of each protocol, allowing you to easily view the source and destination IP addresses, protocol-specific fields, and payload data.
* **Following Streams** for connection-oriented protocols (though less common at pure Layer 3), you can follow streams to see the sequence of packets exchanged between specific logical endpoints.
* **Using Display Filters** create more complex filters to look for packets containing specific IP addresses or within certain IP ranges.

Cloud engineers extract logical identifiers to understand the network topology, communication patterns, and the identities of the communicating devices. However, these are logical identifiers; mapping them to physical machines often requires additional context or correlation with other information like ARP tables or network inventory data.

# Network Topology

Mapping workloads to physical machines requires additional context or correlation with other information like ARP tables or network inventory data. By examining traffic information on end devices and network infrastructure, we gain insights into the physical machines present on the data center network. Although it's often used in conjunction with other methods for more comprehensive asset identification, layer 3 protocols, particularly ARP, helps to identify machines on a data center network. ARP's primary function is to resolve IP addresses to MAC addresses within the same local network segment (Layer 2 broadcast domain).

## Relevant Layer 3 Protocols

Layer 3 protocols primarily deal with logical addressing, ARP provides a crucial link to the physical MAC address of a network interface.

### Routing Protocols (e.g., OSPF, BGP, RIP)

These protocols are used by routers to exchange routing information. Their packet payloads contain details about network topology, advertised routes, and neighbor relationships, which inherently include logical network identifiers (IP addresses of routers, network prefixes). Analyzing these packets can reveal the logical organization of the network.  

### Multicast Protocols (e.g., IGMP, PIM)

* **IGMP (Internet Group Management Protocol)** is used by hosts to join and leave multicast groups. IGMP messages contain the multicast group IP address (a logical identifier for a group of hosts). Examining IGMP reports and queries in a PCAP file shows which hosts are interested in which multicast groups.
* **PIM (Protocol Independent Multicast)** os used by routers to manage multicast traffic. PIM packets contain information about multicast sources and groups, again involving logical IP addresses.
* **Network Management Protocols (e.g., SNMP)** operate at the Application Layer (Layer 7), it often relies on UDP (Layer 4) and ultimately IP (Layer 3) for transport. The payload of SNMP packets (PDUs - Protocol Data Units) contains managed object information, which frequently includes logical identifiers like IP addresses, interface names, and system descriptions. Analyzing SNMP Get, Set, and Trap packets can reveal these identifiers associated with network devices.
* **Tunneling Protocols (e.g., GRE, IPsec)** encapsulate other packets within IP packets to create tunnels. The outer IP header contains the source and destination IP addresses of the tunnel endpoints (logical identifiers of the devices establishing the tunnel). Examining the encapsulated payload might reveal further logical identifiers from the original packets.

### ARP (Address Resolution Protocol)

ARP is responsible for resolving IP addresses (Layer 3 addresses) to MAC addresses (Layer 2 addresses) within a local network. Although it helps in Layer 3 communication by finding the necessary Layer 2 address, ARP itself operates at the boundary between Layer 2 and Layer 3. Its messages are encapsulated directly within Layer 2 frames (like Ethernet frames) and do not contain Layer 3 headers. However, its function is crucial for IP (Layer 3) to deliver packets on a local network. Therefore, it's generally considered a Layer 3 protocol or sometimes referred to as a Layer 2.5 protocol due to its position and function.  

### ICMP (Internet Control Message Protocol)

ICMP is an integral part of the IP protocol suite and is used for error reporting and network diagnostics. ICMP messages are encapsulated within IP packets (Layer 3). While it operates closely with IP, providing feedback about network conditions, it is considered a Layer 3 protocol. Common utilities like ping and traceroute utilize ICMP messages.  

## Logical Identifiers

Layer 3 protocols like IP assign logical addresses to network interfaces. While an IP address isn't tied to the physical hardware permanently (it can change via DHCP or static configuration), it serves as a temporary identifier for a machine on the network. By knowing the IP address of a device, you can interact with it using other Layer 3 protocols like ICMP (for pinging and basic reachability tests) or higher-layer protocols like SSH or SNMP for more detailed querying.

### IP-to-MAC Address Mapping

When a device on the network needs to communicate with another device on the same subnet using its IP address, it uses ARP to find the corresponding MAC address. The requesting device sends out an ARP request containing the target IP address, and the device with that IP address responds with its MAC address. This mapping is stored in the ARP cache of the requesting device.  

### ARP Caches on Network Devices

Network devices like switches and routers also maintain ARP caches. By inspecting the ARP cache of these devices, you can see the recent IP-to-MAC address mappings for the devices connected to that segment of the network.  

### Identifying the Network Interface

The MAC address is a physical address assigned to the Network Interface Card (NIC) of a machine. While not a unique identifier for the entire physical machine (a machine can have multiple NICs), it uniquely identifies a specific network connection.  

### Vendor Identification

The first few octets of a MAC address (Organizationally Unique Identifier - OUI) identify the manufacturer of the NIC. This can sometimes provide clues about the type or vendor of the physical hardware.  

### Network Discovery Tools

Many network scanning and discovery tools utilize ARP to identify active devices on a local network and retrieve their MAC addresses along with their IP addresses and sometimes hostnames. Tools like arp -a (on most operating systems), nmap, and various commercial network management platforms leverage ARP for this purpose.  

## Noise and Anomalies

Working with logical identifier, we must also consider the noise and anomalies that can occur in a data center network. For example, duplicate IP addresses, rogue devices, or misconfigured network interfaces can lead to incorrect mappings or unexpected behavior. Network monitoring tools can help identify these issues by analyzing traffic patterns and device behavior.

### Network Segmentation and Subnets

Data centers often use subnetting to logically group machines based on function, security zone, or other criteria. The IP address range of a subnet can give you a general idea of the physical location or role of the machines within that segment.

### Routing Information

Examining routing tables on network devices (routers, Layer 3 switches) can reveal the paths taken by network traffic. While not directly identifying a physical machine, the routing information can help you understand the network topology and potentially infer the location of certain IP address ranges.  

### DHCP Information

If DHCP is used, the DHCP server maintains a lease database that maps IP addresses to the MAC addresses of the network interfaces that requested them. This provides a link between the logical IP address and the physical MAC address at a specific point in time.


## Limitations and Considerations

### Local Network Segment

ARP only works within the same Layer 2 broadcast domain. You cannot use ARP to directly find the MAC address of a machine on a different subnet; a router would need to be involved.
### Dynamic IP Addresses

If machines use DHCP, their IP addresses can change, making IP-based identification less persistent. However, the MAC address associated with the IP at a given time remains the same for that network interface.  

### MAC Address Spoofing

It's possible for a machine to spoof its MAC address, which could lead to incorrect identification.  

### Virtualization

In virtualized environments, multiple virtual machines might share the same physical NIC or have virtual MAC addresses, making direct physical machine identification via MAC address more complex.

### Management Access Required

To inspect ARP caches on network devices, you typically need administrative access to those devices.


Sources and related content
