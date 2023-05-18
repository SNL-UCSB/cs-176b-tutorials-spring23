# Tutorial 5: P4 + Runtime commands

The tutorial contains the topo directory which creates a very simple topology as show below:
`h1 <--> s1 <--> h2`
where `h1`, `h2` are P4Hosts and `s1` is the P4Switch
* `h1` has the IP address: `10.0.1.1`, mac address: `08:00:00:00:01:11`
* `h2` has the IP address: `10.0.2.2`, mac address: `08:00:00:00:02:22`

Steps:

1) We will write a simple p4 program that forwards packets based on longest prefix match.
2) After we are done writing the P4 program, we will run `make run` in the tutorial-5 directory and a mininet session will start.
3) We will run `pingall 1`(hosts shouldn't be able to communicate with each other). 
4) Next, we will open the switch CLI using the cmd `simple_switch_CLI --thrift-port 9090`, and then we will install the following runtime commands:
```
table_add MyIngress.ipv4_lpm MyIngress.ipv4_forward 10.0.1.1/32 => 08:00:00:00:01:11 1
table_add MyIngress.ipv4_lpm MyIngress.ipv4_forward 10.0.2.2/32 => 08:00:00:00:02:22 2
```
5) We will run `pingall 1` again. What should happen? 
