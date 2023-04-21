# Tutorial 2: OpenFlow + OVS

This tutorial is about OpenFlow + Open VSwitch. Today we will be creating a Open VSwitch, virtual interfaces and attach two hosts(linux namespaces) to the switch(just like in tutorial 1). Then we will tests the connections. After that we will be populating switch with flow rules to change the way network traffic behaves.

![ALT TEXT](https://github.com/SNL-UCSB/cs-176b-tutorials-spring23/blob/main/tutorial1/image.png?raw=true)


## Steps

### List network namespaces.
The command lists all the network namespaces on the system
```
ip netns list
```
### Create 2 VRFs(name spaces) VRF1 and VRF2.
These commands create two network namespaces named VRF1 and VRF2, respectively. Network namespaces are a Linux kernel feature that allows creating multiple virtual network stacks that operate independently and are isolated from each other.

```
ip netns add VRF1
ip netns add VRF2
```
Now new VRFs(name spaces) can be seen.
```
ip netns list
```
### Create virtual interfaces.

Below commands create two pairs of virtual Ethernet interfaces (veth pairs). Each pair consists of two interfaces: veth1 and veth2, and veth3 and veth4 respectively.

When data is sent through one interface of the pair (veth1 and veth3), it is received by the other interface (veth2 and veth4). These virtual interfaces can be used for various networking purposes, such as connecting containers or virtual machines to the network.
```
ip link add veth1 type veth peer name veth2
ip link add veth3 type veth peer name veth4
```
Now the below command will list the interfaces.
```
ip link
```
### Move veth1 and veth3 to VRF1 and VRF2 respectively.

These commands move the virtual interfaces veth1 and veth3 to the respective network namespaces VRF1 and VRF2. The ip link set command is used to modify the attributes of a network interface, and the netns option specifies the target network namespace.
```
ip link set veth1 netns VRF1
ip link set veth3 netns VRF2
```
### Execute commands inside the VRFs to see the interface.
The below commands are used to execute the command `ip link` inside the network namespaces VRF1 and VRF2, respectively. The ip link command is used to display information about network interfaces and their properties.

Executing the command inside each network namespace will show the interfaces available only inside that namespace. This allows you to view the network interfaces that were created earlier with the ip link add command and moved to their respective network namespaces with the ip link set command.
```
ip netns exec VRF1 ip link
ip netns exec VRF2 ip link
```
### Assign IP addresses to VRFs
These commands assign IP addresses to the virtual interfaces veth1 and veth3 that were moved to the VRFs VRF1 and VRF2 respectively.

Assigns the IP address 10.10.10.1 with a netmask of 24 bits to the interface veth1 in the namespace VRF1 and brings it up.

```
ip netns exec VRF1 ifconfig veth1 10.10.10.1/24 up
```
Assigns the IP address 10.10.10.2 with a netmask of 24 bits to the interface veth3 in the namespace VRF2 and brings it up.
```
ip netns exec VRF2  ifconfig veth3 10.10.10.2/24 up
```
### Verify if IP is assigned.
Below command is used to display the network configuration of the veth1 interface inside the VRF1 network namespace. This command is executed inside the VRF1 network namespace using the ip netns exec command.
```
ip netns exec VRF1 ifconfig
```
### Try to ping one VRF form other.
```
ip netns exec VRF1 ping 10.10.10.2
```
Ping is not successful because there is currently no connection between two hosts.
### Create Open vSwitch.
Below command creates a new Open vSwitch bridge named vSwitch1. The "fail-mode=secure" option in the Open vSwitch configuration sets the bridge's behavior when an OpenFlow controller is disconnected from the switch. When the fail-mode is set to "secure", the bridge will block all traffic, and it will only allow traffic to flow when it is explicitly permitted by an OpenFlow rule or when a new controller connection is established. This is a safety feature that prevents unauthorized traffic from flowing through the switch in case of a controller failure or disconnection.

```
ovs-vsctl add-br vSwitch1 -- set Bridge vSwitch1 fail-mode=secure
```
### Show switch.
Below command shows the current configuration of the Open vSwitch, including the switches, bridges, ports, and their status. It lists all the Open vSwitches configured on the system along with their respective bridge, controller, and port configuration information.
```
ovs-vsctl show
```
### Connect virtual interfaces with Open vSwitch
These commands add the two virtual Ethernet interfaces (veth4 and veth2) as ports to the Open vSwitch bridge vSwitch1. This will allow traffic to flow between the two interfaces and the bridge.
```
ovs-vsctl add-port vSwitch1 veth4 -- set Interface veth4 ofport_request=4
ovs-vsctl add-port vSwitch1 veth2 -- set Interface veth2 ofport_request=2
```
Show switch again. you will see the new ports added.
```
ovs-vsctl show
```
### Bring up the virtual interfaces.
Activate the network interfaces veth4 and veth2, respectively.
```
ovs-ofctl mod-port vSwitch1 veth2 up
ovs-ofctl mod-port vSwitch1 veth4 up
```
### Check connectivity.

What should happen?
```
ip netns exec VRF1 ping 10.10.10.2
```

### Forward traffic
These two commands add OpenFlow rules to the OVS (Open vSwitch) switch named vSwitch1. The first command adds a rule that matches packets coming from veth4 as the input port with a priority of 10 and forwards them to veth2. The second command adds a rule that matches packets coming from veth2 as the input port with a priority of 10 and forwards them to veth4. These rules essentially create a bidirectional communication channel between the two veth interfaces by forwarding packets between them through the OVS switch.
```
ovs-ofctl add-flow vSwitch1 "table=0, priority=10, in_port=4, actions=2"
ovs-ofctl add-flow vSwitch1 "table=0, priority=10, in_port=2, actions=4"
```
### Check connectivity.
What should happen now?
```
ip netns exec VRF1 ping 10.10.10.2
```

### Drop ICMP
Below command adds a flow rule to table 0 of the Open vSwitch named vSwitch1. The rule has a priority of 20, which means it has a lower priority than other rules with a higher number. The rule matches any ICMP traffic coming into the switch from the veth4 port and drops it. ICMP packets are used by network devices such as routers and hosts to communicate with each other about network conditions and to diagnose network problems.
```
ovs-ofctl add-flow vSwitch1 "table=0, priority=20, icmp in_port=4, actions=drop"
```
Below command runs a ping test from the VRF1 network namespace to the IP address 10.10.10.2. The ip netns exec command is used to run a command inside a network namespace.
```
ip netns exec VRF1 ping 10.10.10.2
```
Below command adds another flow rule to table 0 of the Open vSwitch named vSwitch1. The rule matches any traffic with a source IP address of 10.10.10.1 and a destination IP address of 10.10.10.2. It drops this traffic.
```
ovs-ofctl add-flow vSwitch1 "table=0, nw_src=10.10.10.1/32, nw_dst=0.10.10.2/32, actions=drop"
```
Below command runs another ping test from the VRF1 network namespace to the IP address 10.10.10.2
```
ip netns exec VRF1 ping 10.10.10.2
```
Since the previous flow rule drops all traffic from the source IP address 10.10.10.1 to 10.10.10.2, this ping test should fail.
