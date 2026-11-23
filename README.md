# SDN Tutorial: Mininet and the Ryu Controller

A hands-on introduction to Software-Defined Networking. You will build
emulated networks with [Mininet](http://mininet.org/), connect them to a
remote [Ryu](https://ryu-sdn.org/) controller over OpenFlow 1.3, and finish
by programming a firewall through Ryu's REST API.

**Contents**

1. [Installation](#1-installation)
2. [Mininet basics](#2-mininet-basics)
3. [Topologies, switches, controllers and links](#3-topologies-switches-controllers-and-links)
4. [Lab: a REST firewall with Ryu](#4-lab-a-rest-firewall-with-ryu)
5. [Ready-made VM](#5-ready-made-vm)

Tested on Ubuntu 20.04; other Linux distributions should work too.

## 1. Installation

### Mininet

```bash
sudo apt update
sudo apt install python3-pip
sudo apt-get install mininet
```

Python 3 is recommended for current Mininet. The packaged version above can
be old and may not support Python 3; to get the latest version, install from
source instead:

```bash
git clone https://github.com/mininet/mininet
cd mininet
util/install.sh -a
```

Check that Mininet works:

```bash
sudo mn
```

### Ryu

```bash
git clone https://github.com/faucetsdn/ryu.git
cd ryu
pip install .
```

### First run with a remote controller

Start Mininet and point it at a controller on localhost:

```bash
sudo mn --controller=remote,ip=127.0.0.1
```

In a second terminal, from the `ryu` directory, start a learning switch:

```bash
ryu-manager --verbose ryu/app/simple_switch_13.py
```

The switch connects to the controller and the hosts can now reach each other.

## 2. Mininet basics

Start the minimal topology (one controller, one switch, two hosts):

```bash
sudo mn
```

Useful commands at the `mininet>` prompt:

| Command | What it does |
| --- | --- |
| `nodes` | Display all nodes |
| `net` | Display all links |
| `dump` | Dump information about all nodes |
| `h1 ping h2` | Run a command on a host |
| `exit` | Leave Mininet |

For every command-line option:

```bash
mn --help
```

## 3. Topologies, switches, controllers and links

### Topology

```bash
sudo mn --topo=TOPO
```

| `TOPO` | Result |
| --- | --- |
| `minimal` | One switch, two hosts |
| `single,X` | One switch with `X` hosts |
| `linear,X` | `X` switches in a line, one host on each |
| `linear,X,Y` | `X` switches in a line, `Y` hosts on each |
| `tree,X,Y` | A tree of depth `X` with fanout `Y` |

### Switch

```bash
--switch=SWITCH
```

`SWITCH` can be `ovsk` (Open vSwitch, kernel) or `user` (user-space switch).

### Controller

```bash
--controller=CONTROLLER
```

`CONTROLLER` can be `ovsc` or `nox`. To attach the switches to a controller
running outside Mininet, use `remote`:

```bash
sudo mn --controller=remote,ip=127.0.0.1 --topo=linear,3 --switch ovsk,protocols=OpenFlow13
```

`ip` is the address of the remote controller, here the local machine.

### Link bandwidth and delay

```bash
sudo mn --link tc,bw=20,delay=20ms
```

This gives every link in the default topology 20 Mbit/s of bandwidth and
20 ms of delay:

```text
(h1)----20ms----(s1)----20ms----(h2)
```

## 4. Lab: a REST firewall with Ryu

The goal is a firewall that filters flows by source and destination IP
address. The topology is one switch, three hosts and one controller:

```text
        (c0)
         |
(h1)----(s1)----(h2)
         |
        (h3)
```

**Step 1.** Create the topology:

```bash
sudo mn --controller=remote,ip=127.0.0.1 --topo single,3 --mac --switch ovsk,protocols=OpenFlow13
```

**Step 2.** Start Ryu with the firewall application:

```bash
ryu-manager --verbose ryu/app/rest_firewall.py
```

**Step 3.** Enable the firewall on the switch:

```bash
curl -X PUT http://localhost:8080/firewall/module/enable/0000000000000001
```

**Step 4.** Ping between hosts from the Mininet prompt:

```text
mininet> h1 ping h2
```

It fails, because the firewall has no rules yet and blocks everything. You
can confirm that the rule list is empty:

```bash
curl http://localhost:8080/firewall/rules/0000000000000001
```

**Step 5.** Allow ICMP in both directions between `h1` and `h2`:

| Source | Destination | Protocol | Action |
| --- | --- | --- | --- |
| 10.0.0.1/32 | 10.0.0.2/32 | ICMP | allow |
| 10.0.0.2/32 | 10.0.0.1/32 | ICMP | allow |

```bash
curl -X POST -d '{"nw_src": "10.0.0.1/32", "nw_dst": "10.0.0.2/32", "nw_proto": "ICMP"}' http://localhost:8080/firewall/rules/0000000000000001
curl -X POST -d '{"nw_src": "10.0.0.2/32", "nw_dst": "10.0.0.1/32", "nw_proto": "ICMP"}' http://localhost:8080/firewall/rules/0000000000000001
```

**Step 6.** Check the rules, then ping again:

```bash
curl http://localhost:8080/firewall/rules/0000000000000001
```

```text
mininet> h1 ping h2
```

The ping now succeeds, while `h3` is still blocked.

## 5. Ready-made VM

A virtual machine image with Mininet and Ryu already installed, for use with
a hypervisor such as VMware Fusion:

- Image: https://drive.google.com/file/d/1mZp_VjSGQNVyvUVtHF9Rk7SPDG4FrV1G/view?usp=sharing
- User: `nza`, password: `8`

## Contact

Questions: nahmed@danforthcenter.org
