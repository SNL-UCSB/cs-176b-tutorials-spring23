
# Tutorial 1: Open vSwitch

This tutorial is about Open Virtual Switch. Today we will be creating a Open VSwitch, virtual interfaces and attach two hosts(linux namespaces) to the switch. Then we will tests the connections.

![ALT TEXT](https://github.com/SNL-UCSB/cs-176b-tutorials-spring23/blob/main/tutorial1/image.png?raw=true)


## Steps

### List namespaces.

``` 
ip netns list
```

### Create 2 VRFs(name spaces) VRF1 and VRF2.

``` 
ip netns add VRF1
```
``` 
ip netns add VRF2
```

### Create virtual interfaces.

``` 
ip link add veth1 type veth peer name veth2
```
``` 
ip link add veth3 type veth peer name veth4
```

Now the below command will list the interfaces.

```
ip link
```

### Move veth1 and veth3 to VRF1 and VRF2 respectively.

```
ip link set veth1 netns VRF1
```
```
ip link set veth3 netns VRF2
```

### Execute commands inside the VRFs to see the interface.
```
ip netns exec VRF1 ip link
```
```
ip netns exec VRF1 ip link
```

### Assign IP addresses to VRFs
```
ip netns exec VRF1 ifconfig veth1 10.10.10.1/24 up
``` 
```
ip netns exec VRF2  ifconfig veth3 10.10.10.2/24 up
``` 

Verify if IP is assigned.
```
ip netns exec VRF1 ifconfig
```

### Try to ping one VRF form other.
```
ip netns exec VRF1 ping 10.10.10.2
```

Ping is not successful because there is currently no connection between two hosts.

### Create Open vSwitch.

```
ovs-vsctl add-br vSwitch1
```

Show switch.
``` 
ovs-vsctl show
```

### Connect virtual interfaces with Open vSwitch 
```
ovs-vsctl add-port vSwitch1 veth4
```
```
ovs-vsctl add-port vSwitch1 veth2
```

Show switch.
```
ovs-vsctl show
```

### Bring up the virtual interfaces.
```
ifconfig veth4 up
```
```
ifconfig veth2 up
```

### Check connectivity.
```
ip netns exec VRF1 ping 10.10.10.2
```

And it works!!!

### Assign IP address to Open vSwitch.
``` 
ifconfig vSwitch1 10.10.10.3/24 
```

With this you can ping created VRF from your machine.
```
ping 10.10.10.1
```

### Find mac address table
ovs-appctl command is used for for querying and controlling Open vSwitch daemon.

```
ovs-appctl fdb/show vSwitch1
```
