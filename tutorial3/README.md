# Tutorial 3

In this tutorial we will create a basic topology consisting of 2 hosts and a switch just like in the previous tutorials. To do this we will be using mininet. For mininet documentation go [here](http://mininet.org/walkthrough/).



![ALT TEXT](https://raw.githubusercontent.com/SNL-UCSB/cs-176b-tutorials-spring23/main/tutorial3/Screenshot%20from%202023-04-27%2014-02-15.png)

## Writing basic Python script that demonstrates how to create a simple network using the Mininet

### Import necessary modules
from mininet.net import Mininet
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.node import OVSController

### Define a function
```
def emptyNet():
```
### Create Mininet instance
```
net = Mininet( controller=OVSController, waitConnected=True )
```
### Add a controller
```
info( '*** Adding controller\n' )
net.addController( 'c0' )
```
### Add Hosts
```
info( '*** Adding hosts\n' )
h1 = net.addHost( 'h1', ip='10.0.0.1' )
h2 = net.addHost( 'h2', ip='10.0.0.2' )
```
### Add Switch
```
info( '*** Adding switch\n' )
s3 = net.addSwitch( 's3' )
```

### Add Links
```
info( '*** Creating links\n' )
net.addLink( h1, s3 )
net.addLink( h2, s3 )
```

### Start Mininet
```
info( '*** Starting network\n')
net.start()

info( '*** Running CLI\n' )
CLI( net )

info( '*** Stopping network' )
net.stop()
```
### Call the function to create the network
```
if __name__ == '__main__':
    setLogLevel( 'info' )
    emptyNet()
```    
## Some basic mininet commands
Mininet provides a CLI (Command Line Interface) that can be used to interact with the network topology. Here are some basic commands that you can use:

`nodes` - Lists all the nodes in the network.
`links` - Lists all the links in the network.
`net` - Shows the network configuration.
`pingall` - Tests connectivity between all hosts in the network.
`<host> <any command inside host>` - Execute any command inside host. 
`<source_host> ping <destination_host>` - Tests connectivity between the host running the command and the specified destination host.
`dump` - Dumps the state of all the switches and hosts in the network.
`exit` - Exits the Mininet CLI.

These are just a few of the many commands that are available in Mininet. You can find more detailed information about these commands and other Mininet commands in the Mininet [documentation](http://mininet.org/walkthrough/).

## Changing network conditions.
For this part of the tutorial lets play around with the network latency.
### Add Links with custom latencies.
Modify the previously written code to specify delay when adding link.
```
net.addLink( h1, s3, delay='50ms' )
```
### Create the topology again.
```
sudo python emptynet.py 
```
### Check latency.
Check latency by executing the following command in mininet CLI.
```
h1 ping h2
```
There should be a delay of aproximately 50 ms.
# Acknowledgement

The example for this tutorial has been taken from [here](https://github.com/mininet/mininet/tree/master/examples) with some minor changes. Apart from that this is a great resource  that demonstrate different mininet features.
