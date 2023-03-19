# A demo session on SDN - with Mininet and Ryu controller
## Installation
It uses Ubuntu 20.04, however, it should work with other Linux operating system too. 

To install Mininet use the following command
### `sudo apt update`
### `sudo install python3-pip`
### `sudo apt-get install mininet`

Python3 is recomended for the latest version of Mininet.  However, the above command may install an older version of Mininet which may not support Python 3. If you would like the latest version of Mininet, consider installing from source as follows. 

Git clone the recent Mininet repo as follows.

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

The following vm image contains Mininet and Ryu installed, can be used in a virtual environemnt like VM Fusion, user: nza, pass: 8
https://drive.google.com/file/d/1mZp_VjSGQNVyvUVtHF9Rk7SPDG4FrV1G/view?usp=sharing

For any query, contact me at nahmed@danforthcenter.org

## Mininet Turotial

Start a minimal topology
### `sudo mn`

This will create a minimal topology with one controller, one switch, and two hosts

Display all nodes
### `mininet>nodes`
Display all links
### `mininet>net`
Dump information about all nodes
### `mininet>dump`
To exit mininet
### `mininet>exit`

There are many options available to create a SDN topology, for more information 
### `mn help`

To create a custom topology

### `sudo mn --topo=TOPO`

where TOPO can be minimal, linear, X (with X switch and 1 host in each), linear, X,Y (with X switch and Y host in each), tree, X, Y (X depth with Y fanout), single, X (single switch with X hosts). 

To select switch type
### `--switch=SWITCH`

where SWITCH can be ovsk or user type.

To select Controller type
### `--controller=CONTROLLER`

where CONTROLLER could be ovsc, and nox. In case of a remote controller to be connected with the Mininet switches, the option should be remote as follows

### `--controller=remote, ip=127.0.0.1`

where ip option helps to set the IP address of the remote controller, in this case it is the localhost.

