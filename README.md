
![nano-node-peering]

<br/>
### Purpose

This document is intended for anyone looking to understand the nano's peering architecture in an easy to digest format.  


### Definitions

>### **Channel**

>_Channels represent the internal representation of an endpoint and it's port._

>### **Peer**  
>Represents the foundation of any distributed network. Peers in the nano network are responsible for replicating transaction blocks to other peers, such that each peer contain the same set of transaction blocks.

>### **Preconfigured_Peers**
>_Represents a list of peers that aide in establishing a connection to identify peers a node can obtain distributed ledger._

>### **Raw Units**  
>Nano contains a total circulating supply is approximately **1.33x10^37** raw units. General measurement is done in **NANO** which is 1x10^29 raw units (**_1 NANO_**).

>### **Representative**

>_A representative is a type of peer that contains enough direct/indirect weight to vote on >incoming blocks._

<br/>

### Initial Peering

The peering process begins with the initial sending of keepalive messages to the default **preconfigured\_peer**, **_peers.nano.org_**. The node software uses the default peering port defined in the node's configuration file (**peering\_port: 7075**). This preconfigured peer functions as a default peer for new nodes. It provides the new node with a randomly selected set of it's own peers. This list serves as the catalyst for a new node to discover other peers participating in the network.

![nano-node-peering-communication]

The image above depicts the basic process of initial peer discovery. The payload within the keepalive message will contain a list of randomly selected peers that the new peer can query for other peers. Once peers establish a communication channel for each other, subsequent keepalive messages will be sent periodically with list of randomly selected peers.

>**NOTE:**  
>  
>  preconfigured\_peers configuration entry is only read upon node bootup. Changes to this list will require a process restart to take effect.

Subsequent request to **preconfigured\_peers** will only occur, if node has yet to identify peers with enough delegated weight that meets the node operators defined **online\_weight\_minimum** (_Default: **6x10^37 raw units**_) configuration.

#### add\_initial\_peers  

This method will validate and verify a node's previous known peers for reuse. Each previous known peer will go through the **node\_id\_handshake** process by means of sending a **keepalive** message.

![nano-node-add-initial-peers]  

New nodes will simply have an empty list which would require it to establish communication with the list of **preconfigured\_peers** to obtain a list of other peers. This internal process known as **ongoing_crawl** will reach out to all preconfigured peers and send a keepalive message. 

##### node\_id\_handshake process

###### send
The **node\_id\_handshake** message contains two items. The first is a **query** item which is the **syn\_cookie** allocated by the peer. The second is a **response** hash pair which contains a reference to the peer's **node\_id** along with a signed message by the node.

![nano-node-send-node-handshake]

Upon receiving this message, the node will execute it's own **node\_id\_handshake** handler method.

###### process

The handler method will extract the **query** and **response** values from the node\_id\_handshake message. Once values are extracted, handler will determine if message is from existing peer or new peer.


![nano-node-process-node-handshake]

If **existing peer**, verify the following:

- message signature is valid
- handshake is not local
- Verify no existing node id and different port exists, if so remove

Once completed, a new communication channel is created and recorded for this peer, registering its address, port, and node_version.

If **non-existing peer**, assign a syn cookie for this endpoint and start the node handshake communication by sending peer a **node\_id\_handshake** message.

Finally record stats for this handshake occurrence. These steps are repeated for each new peer.




[nano-node-peering]: ./images/node/nano-node-peering.png
[nano-node-add-initial-peers]: ./images/node/nano-node-add-initial-peers.png
[nano-node-peering-communication]: ./images/node/nano-node-peering-communication.png
[nano-node-send-node-handshake]: ./images/node/nano-node-send-node-id-handshake.png
[nano-node-process-node-handshake]: ./images/node/nano-node-process-node-id-handshake.png







