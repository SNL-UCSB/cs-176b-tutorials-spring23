Recall this exercise
![ALT TEXT](https://github.com/SNL-UCSB/cs-176b-tutorials-spring23/blob/main/tutorial1/image.png?raw=true)

The steps that we followed to create a simple topology with two hosts connected to an OVS switch. 

```
ip netns add VRF1
ip netns add VRF2
ip link add veth1 type veth peer name veth2
ip link add veth3 type veth peer name veth4
ip link set veth1 netns VRF1
ip link set veth3 netns VRF2
ip netns exec VRF1 ifconfig veth1 10.10.10.1/24 up
ip netns exec VRF2  ifconfig veth3 10.10.10.2/24 up
...
```

With Mininet, you can get the same outcome with a single command. 
```
sudo mn --switch ovsk --topo single,2
```
This command creates a Mininet network with a single Open vSwitch (OVS) switch, connected to two hosts. The --switch option is set to ovsk, which indicates that the switch type should be Open vSwitch Kernel (OVSK), which is the default switch type in Mininet. 

The --topo argument in Mininet is used to specify the network topology. In this case, --topo single,2 specifies a simple linear topology consisting of a single switch and two hosts, with one host connected to each port of the switch. The single specifies a single switch, and the 2 specifies two hosts.

Now, let's ensure that the OVS is controlled by an OVS controller.
Step 1: 
```
sudo mn --controller remote --switch ovsk --topo single,2
```
Here, we specify that the controller for the network should be `remote`, meaning that it will not be a part of the Mininet network but rather a separate entity outside of the network.

Set the controller for this switch
```
sudo ovs-vsctl set-controller s1 tcp:127.0.0.1:6653
```

What do you expect to see here?
```
sudo ovs-ofctl dump-flows s1
```

Nothing. Try pinging the Mininet hosts. What do you expect to see?

Now, add the flow rules
```
sudo ovs-ofctl add-flow s1 "table=0, priority=10, in_port=s1-eth2, actions=s1-eth1"
sudo ovs-ofctl add-flow s1 "table=0, priority=10, in_port=s1-eth1, actions=s1-eth2"
```
The first command is adding a rule to "table 0" (the initial table in OpenFlow switches) with a priority of 10. This rule specifies that packets received on port "s1-eth2" should be forwarded out through port "s1-eth1".

The second command is adding a similar rule with a priority of 10, but it specifies that packets received on port "s1-eth1" should be forwarded out through port "s1-eth2".

These two rules together create a bidirectional link between "s1-eth1" and "s1-eth2", allowing packets to flow between the two ports.

We should now see this: 
```
vagrant@ubuntu-bionic:~$ sudo ovs-ofctl dump-flows s1
 cookie=0x0, duration=10.150s, table=0, n_packets=0, n_bytes=0, priority=10,in_port="s1-eth2" actions=output:"s1-eth1"
 cookie=0x0, duration=2.698s, table=0, n_packets=0, n_bytes=0, priority=10,in_port="s1-eth1" actions=output:"s1-eth2"
```
Now try pining the Mininiet hosts. Do you observe any change?
 
