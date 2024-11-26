
So I wanted to gain some experience with VLANs and subnets since I've been started to work with Kubernetes to understand better how the networking works. So as I'm developing my homelab I have the necessary infrastructure to learn some networking basics. So here's my experiences.

## Understanding VLANs and Subnets

### What Are VLANs?

A Virtual Local Area Network (VLAN) is a logical segmentation of a physical network. VLANs allow multiple distinct networks to exist on the same physical infrastructure by tagging traffic with VLAN identifiers (VLAN IDs). This segmentation ensures that devices in one VLAN cannot directly communicate with devices in another VLAN unless explicitly configured to do so via a router or Layer 3 switch. VLANs operate at Layer 2 (the Data Link Layer) of the OSI model.

### What Are Subnets?

A subnet is a division of an IP network into smaller, more manageable segments. Subnets help organize and isolate network traffic at Layer 3 (the Network Layer) of the OSI model. Each subnet is identified by a unique IP address range, and communication between subnets requires a router or Layer 3 switch.

### How VLANs and Subnets Are Related

VLANs and subnets often work together:

- **VLANs handle traffic isolation at Layer 2** by tagging packets and ensuring devices on different VLANs cannot communicate directly.
- **Subnets manage traffic isolation at Layer 3** by assigning different IP ranges to each network segment.
- Each VLAN typically maps to a specific subnet to ensure seamless Layer 2 and Layer 3 communication.

### Why Are Both VLANs and Subnets Necessary for Proper Network Isolation?

| Benefit                                     | Role of VLANs                                                                          | Role of Subnets                                                                                     |
| ------------------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Enhanced Security**                       | Prevents devices in one VLAN from communicating with those in another VLAN at Layer 2. | Ensures inter-VLAN communication must go through a router or firewall with enforced security rules. |
| **Traffic Control and Management**          | Limits broadcast domains, reducing unnecessary traffic and improving performance.      | Implements routing policies and access controls for specific IP ranges.                             |
| **Network Scalability**                     | Allows logical grouping of devices regardless of physical location for flexibility.    | Simplifies IP address management and efficient use of IP space.                                     |
| **Reduced Failure Impact**                  | Confines Layer 2 issues (e.g., broadcast storms) to a single VLAN.                     | Isolates Layer 3 issues (e.g., routing misconfigurations) to a specific IP range.                   |
| **Improved Monitoring and Troubleshooting** | Provides visibility into Layer 2 traffic flows within VLAN segments.                   | Facilitates identification and resolution of Layer 3 issues, such as routing errors.                |

### What Is a Trunk Port?

A trunk port is a network switch port configured to carry traffic for multiple VLANs. Unlike access ports, which belong to a single VLAN, trunk ports tag packets with their VLAN ID to allow devices and switches on the network to distinguish traffic from different VLANs. Trunk ports are essential for connecting VLAN-aware devices such as routers, other switches, and hypervisors.

### Why Separate a Network with VLANs and Subnets?

1. **Security:** By isolating traffic, you can limit the exposure of sensitive systems to insecure or untrusted devices.
2. **Performance:** Segmentation reduces broadcast domains, preventing unnecessary traffic from bogging down the network.
3. **Flexibility:** VLANs and subnets allow dynamic and scalable network configurations.
4. **Learning:** Configuring VLANs and subnets provides hands-on experience with enterprise networking concepts.

## Let's do this

After the dry material, I'd like to introduce my little journey. My goal was to separate the networking infrastructure, my local ALM and Kubernetes homelab infra, IoT devices and the other, user devices. Because for security reasons and for fun :-) 

### Current Limitations

I have to mention at this point I wasn't able to achieve my goals because my network is separated into two part. The ISP and my pfSense router are in the other part of my flat than my office with my PC and server. I connected them with a WiFi mesh network, provided by 3 TP-Link Deco X10. This mesh system sadly does not support VLAN tags. It gave me some headache because the fact, that the Deco X10 do not support VLAN tags caused nondeterministic behavior, such as intermittent connectivity. For example I was able sometimes to ping internet addresses or from the default VLAN some other machines, but sometimes I wasn't. So I have to gain some courage in the future and make holes on the walls or buy more advance WiFi mesh devices.

### Steps for VLAN and Subnet Configuration

#### 1. Define VLANs in pfSense

Navigate to **Interfaces > Assignments > VLANs** in pfSense.

- Specify the **Parent Interface** (e.g., your Ethernet interface).
- Assign a **VLAN Tag** (e.g., VLAN Tag 10).
- Add a description (e.g., "IoT devices").
- Optionally, set the VLAN Priority.

**Example:**
VLAN Tag 10 matches the 192.168.10.0/24 subnet.

#### 2. Assign VLANs to Interfaces

Once VLANs are defined, assign them to interfaces:

1. Go to **Interfaces > Assignments.**
2. Select the VLAN from the dropdown menu and click **Add.**
3. Enable the interface, give it a descriptive name (e.g., IoT\_VLAN), and configure:
   - IPv4 Configuration: **Static IPv4**
   - IPv4 Address: Set the gateway IP (e.g., 192.168.10.1/24).
   - Ensure the correct CIDR notation!

#### 3. Configure DHCP

Each VLAN requires its own DHCP settings:

- Go to **Services > DHCP Server.**
- Select the VLAN interface and enable DHCP.
- Assign IP ranges matching the subnet (e.g., 192.168.10.100 to 192.168.10.200).

For static IPs, match MAC addresses with reserved IPs in pfSense.

#### 4. Configure the Switch

Managed switches handle VLAN tagging at Layer 2:

1. Access the switch’s VLAN configuration (e.g., TP-Link’s 802.1Q VLAN submenu).
2. Assign VLANs to specific ports:
   - **Access ports:** For devices within a single VLAN.
   - **Trunk ports:** For connecting to routers or devices managing multiple VLANs.
3. Ensure the router port or upstream switch port is configured as a trunk port.

#### 5. Configure the Hypervisor

For Proxmox:

1. Enable VLAN awareness in the bridge configuration:
   - Go to **System > Network** in the Proxmox UI.
   - Enable **VLAN Aware** on the bridge interface (e.g., vmbr0).
2. Assign VLAN tags to virtual machines:
   - Select the VM in Proxmox.
   - Go to **Hardware > Network Device.**
   - Set the VLAN Tag.
   - Restart the VM to apply changes.


