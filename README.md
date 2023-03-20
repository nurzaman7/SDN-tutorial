# SDN demo - with Mininet and Ryu controller
## Installation
It uses Ubuntu 20.04 but should also work with other Linux operating systems. 

To install Mininet, use the following command.

### `sudo apt update`
### `sudo install python3-pip`
### `sudo apt-get install mininet`

Python3 is recommended for the latest version of Mininet.  However, the above command may install an older version of Mininet which may not support Python 3. If you would like the latest version of Mininet, consider installing from the source as follows. 

Git clones the recent Mininet repo as follows.

### `git clone https://github.com/mininet/mininet`
### `cd mininet`
### `~/mininet/util/install.sh -a`

Run the following command to check Mininet's working

### `sudo mn`

Let's install Ryu controller now.

### `git clone https://github.com/faucetsdn/ryu.git`
### `cd ryu`
### `pip install .`

Let's start the Mininet again as follows:

### `sudo mn --controller=remote, ip=127.0.0.1`

Now go to the ryu directory, execute the following command

### `ryu-manager --verbose ryu/app/simple_switch-13.py`

SDN controler should work now!


## Mininet Turotial

Start a minimal topology
### `sudo mn`

This will create a minimal topology with one controller, one switch, and two hosts.

Display all nodes
### `mininet>nodes`
Display all links
### `mininet>net`
Dump information about all nodes
### `mininet>dump`
To exit mininet
### `mininet>exit`

There are many options available to create a SDN topology, for more information. 
### `mn help`

To create a custom topology

### `sudo mn --topo=TOPO`

where TOPO can be minimal, linear, X (with X switch and 1 host in each), linear, X,Y (with X switch and Y host in each), tree, X, Y (X depth with Y fanout), single, X (single switch with X hosts). 

To select switch type
### `--switch=SWITCH`

where SWITCH can be ovsk or user type.

To select Controller type
### `--controller=CONTROLLER`

where CONTROLLER could be ovsc, and nox. In case of a remote controller to be connected with the Mininet switches, the option should be remote as follows.

### `sudo mn --controller=remote,ip=127.0.0.1 --topo=linear,3 --switch ovsk,protocols=OpenFlow13`


where ip option helps to set the IP address of the remote controller, in this case it is the localhost.


How to set link bandiwth and delay?
### `sudo mn --link tc,bw=20,delay=20ms`

set bandwidh of 20Mb, and delay of 20ms for the link, in the default topology

### `(h1)----20ms----(s1)-----20ms-----(h2)`

## Ryu Controller application
For this, lets create a simple firewall that filters flow based on the destination IP address and the source IP address. For this, we will be using the single topology, which has three hosts, one switch, and one controller.

 
### `       (c0)`
### `        |`
### `(h1)----(s1)-----(h2)`
### `        |`
### `       (h3)`
 
Lets create a Mininet topology
### `sudo mn --controller=remote,ip=127.0.0.1 --topo single,3 --mac --switch ovsk,protocols=OpenFlow13`


We will be using Ryu controller, lets start the controller,
### `ryu-manager --verbose ryu/app/rest_firewall.py`

To eanble firewall

### `curl -X PUT http://localhost:8080/firewall/module/enable/0000000000000001`

lets ping hosts
### `h1 ping h2`
It should not work, as there is no rules set yet.

### `curl http://localhost:8080/firewall/rules/0000000000000001`

To eanble ping, we can set firewall rules, as follows:

### `source: 10.0.0.1/32, destination: 10.0.0.2/32, protocol: ICMP, and permission: allow`
### `source: 10.0.0.2/32, destination: 10.0.0.1/32, protocol: ICMP, and permission: allow`

curl commands would be

### `curl -X POST -d '{"nw_src": "10.0.0.1/32", "nw_dst": "10.0.0.2/32", "nw_proto": "ICMP"}' http://localhost:8080/firewall/rules/0000000000000001`
### `curl -X POST -d '{"nw_src": "10.0.0.2/32", "nw_dst": "10.0.0.1/32", "nw_proto": "ICMP"}' http://localhost:8080/firewall/rules/0000000000000001`

check the rules

### `curl http://localhost:8080/firewall/rules/0000000000000001`

Ping should work now!!

The following vm image contains Mininet and Ryu installed and can be used in a virtual environment like VM Fusion, 
user: nza, pass: 8
https://drive.google.com/file/d/1mZp_VjSGQNVyvUVtHF9Rk7SPDG4FrV1G/view?usp=sharing

For any query, contact me at nahmed@danforthcenter.org
