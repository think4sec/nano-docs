**Notes to gain understanding of nano architecture. A lot of shortcuts taken to quickly digest some of the code. Will need to refactor layout and information once more information is digested.**  

**Section focuses on peering**

# Peering

###  [Overview]()

### [Configuration]()
* [allow\_local\_peers]()
* [peering_port]()
* [preconfigured_peers]()
* [preconfigured_representatives]()
* [work_peers]()

###  [Types]()

* [Standard]()
* [Representative]()
* [Work]()

### [Initialization]()
  * [add\_initial\_peers]()
  * [network::start]()
  * [ongoing\_peer\_store]()  
  * [rep\_crawler::start]()
  * [store() (**_mdb_**)]()
  
### [Start]()
  * [node::start()]()
     * [network::start]()
     * [add\_initial\_peers]()
     * [rep\_crawler::start]()
     * [ongoing\_peer\_store]() 
     * [ongoing\_rep\_calculation]()
     * [ongoing\_online\_weight\_calculation\_queue]() 
     * [port\_mapping::start]()
     
### [Running]()

   * [broadcast\_confirm\_req]()
   * [broadcast\_confirm\_req\_base]()
   * [keepalive]()
   * [merge_peers]()
   * [send\_keepalive]()
   * [send\_node\_id\_handshake]()
     

## Overview




## Configuration
### Allow\_Local\_Peers:

This option allows a node operator to peer with rfc1918 related nodes. Otherwise, reserved addresses are restricted. Default is **_false_**.

### Peering\_Port:

Main port used to publish/receive blocks to/from other peers. Default port **_7075_**

### Preconfigured\_Peers:

This entry is used to define an initial list of peers node should use. Currently the default is **_peering.nano.org_**

### Preconfigured\_Representatives:



### Work Peers:

A json array containing pairing of **host and port** destinations where proof-of-work can be submitted

## Types

## Initialization
![Daemon Initialize][daemon-init]

### network::start


![node-network-init]

## Start
![Peering Related Startup][peering-boot]

### network::start:

Main entry for majority of functionality for node. It setups listener for peering as well as sockets (**channels**) for peer communication.

![node-network-start]

![node-network-udp-start]
[node-network-udp-receive]

### add\_initial\_peers:

The purpose of this method appears to be related to the setup of an initial communication list for the node. 

The image below depicts the main sequence taken when peers are listed within a node's configuration file. Each peer is validated by requesting a vote for a random block. Peer has a total of **5** seconds to issue vote, otherwise peer is discarded and method iterates to the next entry in the list.  

![Add Initial Peers][add-initial-peers]


### rep\_crawler::start

![Rep Crawler::start][rep-crawler-start]

### ongoing\_peer\_store

![Ongoing Peer Store][node-ongoing-peer-store]

## Running

### network:

![node-network-udp-process-packets]
![node-network-udp-receive-action]

[peering-overview]: ../images/overview/peering-overview.png
[daemon-init]: ../images/boot/daemon-init.png

[peering-boot]: ../images/node/node-peering-boot.png

[add-initial-peers]: ../images/start/add-initial-peers.png
[rep-crawler-start]: ../images/node/rep_crawler/rep-crawler-start.png
[node-ongoing-peer-store]: ../images/node/node-ongoing-peer-store.png

[node-network-start]: ../images/node/network/network-start.png
[node-network-init]: ../images/node/network/network-init.png
[node-network-udp-process-packets]: ../images/node/network/transport/udp/transport-udp-channels-process-packets.png
[node-network-udp-start]: ../images/node/network/transport/udp/transport-udp-channels-start.png
[node-network-udp-receive]: ../images/node/network/transport/udp/transport-udp-channels-receive.png
[node-network-udp-receive-action]: ../images/node/network/transport/udp/transport-udp-channels-receive-action.png

[store-init]: https://github.com/nanocurrency/nano-node/blob/d0153243f9d3f89e34b211ee566c1100e502fa3a/nano/node/node.cpp#L1020
[network-init]: https://github.com/nanocurrency/nano-node/blob/d0153243f9d3f89e34b211ee566c1100e502fa3a/nano/node/node.cpp#L1026