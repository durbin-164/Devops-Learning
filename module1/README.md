1. Install virtual box and vagrant
2. Create a folder, example module1
3. vagrant init ubuntu/bionic64
4. vagrant up
5. vagrant ssh
6. sudo ip netns add ns1
7. sudo ip netns add ns2
8. sudo ip link add veth1 type veth peer name vpeer1
9. sudo ip link add veth2 type veth peer name vpeer2
10. sudo ip link set veth1 netns ns1
11. sudo ip link set veth2 netns ns2
12. sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1
13. sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2
14. sudo ip netns exec ns1 ip link set veth1 up
15. sudo ip netns exec ns2 ip link set veth2 up
16. Now set up bridge
17. sudo ip link add br1 type bridge
18. sudo ip link set br1 up
19. sudo ip link set vpeer1 master br1
20. sudo ip link set vpeer2 master br1
21. sudo ip addr add 10.10.0.1/24 dev br1
22. sudo ip netns exec ns1 ip route add default via 10.10.0.1
23. sudo ip netns exec ns2 ip route add default via 10.10.0.1
24. sudo ip netns exec ns1 ip link set lo up
25. sudo ip netns exec ns2 ip link set lo up
26. sudo ip netns exec ns2 ping 10.10.0.2
27. sudo ip netns exec ns1 ping 10.10.0.3

# Setup env
Initialize Vagrant with Ubuntu Bionic64 box
```bash
# Initialize Vagrant
vagrant init ubuntu/bionic64

# Start the virtual machine
vagrant up

# Connect to the virtual machine
vagrant ssh
```


# Network Namespace and Bridge Configuration

This guide provides step-by-step instructions for setting up network namespaces and a bridge interface using the `ip` command-line tool.

## Group 1: Network Namespace and veth Interfaces

```bash
sudo ip netns add ns1
sudo ip netns add ns2
sudo ip link add veth1 type veth peer name vpeer1
sudo ip link add veth2 type veth peer name vpeer2
sudo ip link set veth1 netns ns1
sudo ip link set veth2 netns ns2
```

These commands create two network namespaces named ns1 and ns2 and four veth interfaces (veth1, vpeer1, veth2, vpeer2). Network namespaces provide isolated network environments, and veth interfaces are used to connect the namespaces.


# Group 2: IP Address Assignment and Interface Configuration
```bash
sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1
sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2
sudo ip netns exec ns1 ip link set veth1 up
sudo ip netns exec ns2 ip link set veth2 up
```

These commands assign IP addresses to the veth interfaces within their respective network namespaces. Bringing up the veth interfaces ensures that they are active and ready for communication.

# Group 3: Bridge Configuration
```bash 
sudo ip link add br1 type bridge
sudo ip link set br1 up
sudo ip link set vpeer1 master br1
sudo ip link set vpeer2 master br1
sudo ip addr add 10.10.0.1/24 dev br1
```

The above commands create a bridge interface named br1. The bridge interface acts as a virtual switch that connects the veth interfaces. By associating the vpeer interfaces with the bridge, packets can be forwarded between the namespaces. The IP address is assigned to the bridge interface to enable communication with the connected namespaces.


# Group 4: Network Namespace Routing Configuration
```bash
sudo ip netns exec ns1 ip route add default via 10.10.0.1
sudo ip netns exec ns2 ip route add default via 10.10.0.1
```

These commands configure the default routes for the network namespaces ns1 and ns2. The default route specifies the gateway IP address to forward packets to destinations outside the namespace. In this case, the gateway is the bridge interface (br1).


# Group 5: Loopback Interface Configuration
```bash
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns2 ip link set lo up
```

If the loopback interfaces in the network namespaces (ns1 and ns2) are not brought up, the commands in steps 26 and 27 would likely fail.

The ping command is used to send ICMP echo request packets to a destination IP address and expects to receive ICMP echo reply packets in response. When you ping an IP address within the same network namespace, the ICMP packets are sent through the loopback interface.

If the loopback interfaces are not up, the ICMP packets sent by the ping command would not be able to reach the destination IP address (10.10.0.2 in step 26 or 10.10.0.3 in step 27) within the network namespace. As a result, the ping command would not receive any reply packets and would indicate a failure.

To ensure successful ping commands between the IP addresses within the network namespaces, it is necessary to have the loopback interfaces brought up.


# Group 6: Testing Connectivity
```bash
sudo ip netns exec ns2 ping 10.10.0.2
sudo ip netns exec ns1 ping 10.10.0.3
```
These commands test connectivity by pinging IP addresses between the network namespaces. It ensures that the namespaces are properly configured and able to communicate with each other.

Make sure to run the commands in the provided order to set up the network namespaces and bridge interface correctly.


# All in one

```bash
#!/bin/bash

# Install virtual box and vagrant
# [Instructions for installing virtual box and vagrant]

# Create a folder
mkdir module1
cd module1

# Initialize Vagrant
vagrant init ubuntu/bionic64

# Start Vagrant
vagrant up

# SSH into the Vagrant box
vagrant ssh

# Create network namespaces
sudo ip netns add ns1
sudo ip netns add ns2

# Create veth interfaces
sudo ip link add veth1 type veth peer name vpeer1
sudo ip link add veth2 type veth peer name vpeer2

# Move veth interfaces to respective namespaces
sudo ip link set veth1 netns ns1
sudo ip link set veth2 netns ns2

# Assign IP addresses to veth interfaces
sudo ip netns exec ns1 ip addr add 10.10.0.2/24 dev veth1
sudo ip netns exec ns2 ip addr add 10.10.0.3/24 dev veth2

# Bring up veth interfaces
sudo ip netns exec ns1 ip link set veth1 up
sudo ip netns exec ns2 ip link set veth2 up

# Set up the bridge
sudo ip link add br1 type bridge
sudo ip link set br1 up
sudo ip link set vpeer1 master br1
sudo ip link set vpeer2 master br1
sudo ip addr add 10.10.0.1/24 dev br1

# Configure routing in network namespaces
sudo ip netns exec ns1 ip route add default via 10.10.0.1
sudo ip netns exec ns2 ip route add default via 10.10.0.1

# Bring up loopback interfaces
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns2 ip link set lo up

# Test connectivity
sudo ip netns exec ns2 ping 10.10.0.2
sudo ip netns exec ns1 ping 10.10.0.3
```