
![nano-node-peering]

<br/>  

+ [Purpose]()  
+ [Definitions]()  
	* [channel]()  
	* [peer]()  
	* [preconfigured\_peers]()  
	* [raw\_units]()  
	* [representative]()  
+ [Message Types]()  
	* [confirm\_ack]()  
		- [send]()
		- [receive]()
	* [confirm\_req]()  
		- [send]()
		- [receive]()
	* [keepalive]()
		- [send]()
		- [receive]()
	* [node\_id\_handshake]()  
		- [send]()
		- [receive]()
	* [publish]()  
+ [Initial Peering]()
	* [add\_initial\_peers]()  
	* [rep\_crawler]()  
+ [Continous Peering]()
	* [rep\_crawler thread]()
	* [ongoing\_peer\_store]()
	
<br/>  

### Purpose

This document is intended for anyone looking to understand the nano's peering architecture in an easy to digest format.  


### Definitions

>### **channel**

>_Channels represent the internal representation of an endpoint and it's port._

>### **peer**  
>Represents the foundation of any distributed network. Peers in the nano network are responsible for replicating transaction blocks to other peers, such that each peer contain the same set of transaction blocks.

>### **preconfigured_peers**
>_Represents a list of peers that aide in establishing a connection to identify peers a node can obtain distributed ledger._

>### **raw units**  
>Nano contains a total circulating supply is approximately **1.33x10^37** raw units. General measurement is done in **NANO** which is 1x10^29 raw units (**_1 NANO_**).

>### **representative**

>_A representative is a type of peer that contains enough direct/indirect weight to vote on >incoming blocks._

>### **syn_cookies**
>

<br/>

### Message Types  

Section covers the different message types currently used for peer communication. Each message type, contains an appropriate message header. The message header will contain the following; **version\_max**, **version\_using**, **version\_min**, **payload message type**, and **extension**.  

Size of header is as follows:

| Member | Size | Description
--- | --- | ---  
version_max | 1 byte | Max version of node  
version_using | 1 byte | Current version of node  
version_min | 1 byte | Min version of node
type | 1 byte | Hex code representation of payload message type.
 &nbsp; | &nbsp; | invalid = 0x0
 &nbsp; | &nbsp; | not\_a\_type = 0x1
 &nbsp; | &nbsp; | keepalive = 0x2
 &nbsp; | &nbsp; | publish = 0x3
 &nbsp; | &nbsp; | confirm\_req = 0x4
 &nbsp; | &nbsp; | confirm\_ack = 0x5
 &nbsp; | &nbsp; | bulk\_pull = 0x6
 &nbsp; | &nbsp; | bulk\_push = 0x7
 &nbsp; | &nbsp; | frontier\_req = 0x8
 &nbsp; | &nbsp; | node\_id\_handshake = 0x0a
 &nbsp; | &nbsp; | bulk\_pull\_account = 0x0b  
 extensions | 2 bytes | <need to review further>  
 block\_type\_mask | 2 bytes | <need to review further>  
 
Total size of header = **8 bytes**  

Each message will contain a **header** member that can be used to set/fetch header related information. In addition, each message sent will send a **header\_magic\_number**, this value represents the network node is sending messages for. The size of this magic number is **2 bytes**.

**Dynamic payload size based on type**  

| Type | Min. | Max | Description   
--- | --- | --- | --- 
bulk\_pull | 0 | 0 | 
bulk\_pull\_account | 0 | 0 | 
bulk\_push | 0 | 0 | 
frontier\_req | 0 | 0 
keepalive | 0 | 0 | 

#### confirm\_ack  

The purpose of this message type is to send **votes** to other peers for specific blocks.

The message contains the following members:  

| Member | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
vote | &nbsp; | A set of votes  

A vote object contains the following members:

| Member | Size | Description
--- | --- | ---  
sequence | 8 bytes | Vote round sequence number
blocks | &nbsp; | Blocks or Block Hashes vote intended for
account | 32 bytes | Account that is voting  
signature | 64 bytes | Signature of **sequence** + **blocks**
hash_prefix | &nbsp; | &nbsp;

#### confirm\_req  

The purpose of this message type is to request **votes** from other peers for specific blocks. 

The message contains the following members:  

| Member | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
block | &nbsp; | Block transaction
roots\_hashes | 64*total pairs in set | A set of Block's \<hash,root\> pair. <br/>**hash** = digest of hashables in block. <br/>**root** = previous block or account number for open blocks

###### send  

###### receive

The message will either contain the **block** or **roots_hashes** member populated. 

![nano-node-confirm-req-recv-overview]

**Contain Block**

![nano-node-confirm-req-recv-contain-block]  

**Contain Root Hashes**  

![nano-node-confirm-req-recv-contain-hashes]

#### keepalive  

The purpose of this message type is to send a set of a node's **peers** to other **peers**. Peers are randomly selected and packed into this message object.

The message contains the following data:

| Member | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
peers | 144 bytes | Array holding upto 8 nano::endpoint type objects

Storage allocation: 

(Total nano::endpoint object stored)*(ipv6\_addr\_len+port\_len) = Total size

8 elements = Total stored nano::endpoint objects  
16 bytes = ipv6\_addr\_len  
2 bytes = port\_len  

8\*(16+2) = **144 bytes**

###### send  

![nano-node-send-keepalive]  

###### receive  

The receive handler for keepalive messages will first check sending peer has not breached the node's threshold for max connections per ip (**max\_peers\_per\_ip** = 10). If breach occurs, stats are simply updated for received message. If breach does not occur, generate cookie and store for peer.   

![nano-node-process-keepalive]  

#### node\_id\_handshake  

The node\_id\_handshake allows for node's to uniquely identify one another. The message stores the following data:

| Member | Length | Description  
--- | --- | ---  
header | 8 bytes | Message header
query | 32 bytes | Reference syn_cookie created for peer
response | 64 bytes | Hash containing node's public_id and signature  


###### send

Sending this message type, the node will first allocate a **syn\_cookie** for peer as well signature of message. This signature will be used to verify validity of message. Both **query** and **response** data are populated, before sending message to peer.

![nano-node-send-node-handshake]  


###### receive

The handler will deserialize message and use the **query** and **response** data. Message signature is check for validity, otherwise node will start node\_id\_handshake process with peer.

![nano-node-process-node-handshake]

If **existing peer**, verify the following:

- message signature is valid
- handshake is not local
- Verify no existing node id and different port exists, if so remove

Once completed, a new communication channel is created and recorded for this peer, registering its address, port, and node_version.

If **non-existing peer**, assign a syn cookie for this endpoint and start the node\_id\_handshake process sending a **node\_id\_handshake** message to peer.

Finally record stats for this handshake occurrence.  

#### publish  

This message type is used to **publish** blocks to network.

This message contains the following members:  

| Members | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
block | &nbsp;| Block transaction

A block contains the following members:

| Members | Size | Description
--- | --- | ---  
hashables | &nbsp; | Represents block's associated account (public key), <br/>previous hash in chain, account representative, balance, link field,<br/> size = <br/>(summation for sizeof account+previous+representative+balance+link)
signature | 64 bytes | Represents signature of block
work | 8 bytes | Reprsents compute work for block
size | &nbsp; | represents summation for sizeof hashables+signature+work


### Initial Peering

The peering process starts by identifying and adding any existing peers stored in data store. If their are no existing peers, the internal process **rep\_crawler** will reach out to **preconfigured\_peers**. These preconfigured\_peers are used as seed peers to identify other network participants. 

The default **preconfigured\_peer** is **_peers.nano.org_** . Communication starts by sending a **keepalive** message (_See [keepalive]() message type above_). By design, keepalive messages will attempt to supply any peer with a random selection of it's own peers. Thus the sending of keepalive messages is the building blocks for nodes to establish a near complete list of peers. 

The image below depicts the initial communication flow to preconfigured peers.
 
![nano-node-peering-communication]


>**NOTE:**  
>  
>  preconfigured\_peers configuration entry is only read upon node bootup. Changes to this list will require a process restart to take effect.

The purpose of the **rep\_crawler** process is to establish communication with as many peers to meet the node's defined **online\_weight\_minimum** (_Default: **6x10^37 raw units**_).

#### add\_initial\_peers  

This method will validate and verify a node's previous known peers for reuse. Each previous known peer will go through the **node\_id\_handshake** process.

![nano-node-add-initial-peers]  

New nodes will simply have an empty list which would require it to establish communication with the list of **preconfigured\_peers** to obtain a list of other peers.  

#### rep\_crawler  

Rep\_crawler will run every **7 seconds** if node contains sufficient weight. Otherwise, run-time is every **3 seconds** until node has communicated with a sufficient amount of weighted peers. The total weight of it's peers must be greater than the **online\_weight\_minimum** configuration entry. 

Upon each execution, rep\_crawler attempts to poll last known weighted peers prior to selecting a random set of peers. The size of random peers to poll range from **15** to **60**. This range is determined by a nodes' peer's weight. Once a random list of peers are chosen, each peer is polled and sent a **confirm\_req** message. This message will contain a random block selected from the ledger. Peers are given up to **5 seconds** to respond. If no response is given, block is simply removed from queue. 

![nano-node-repcrawler-ongoing-crawl]

[nano-node-confirm-req-recv-contain-block]: ./images/node/nano-node-confirm-req-recv-contain-block.png
[nano-node-confirm-req-recv-contain-hashes]: ./images/node/nano-node-confirm-req-recv-contain-hashes.png
[nano-node-confirm-req-recv-overview]: ./images/node/nano-node-confirm-req-recv-overview.png
[nano-node-peering]: ./images/node/nano-node-peering.png
[nano-node-send-keepalive]: ./images/node/nano-node-send-keepalive.png
[nano-node-process-keepalive]: ./images/node/nano-node-process-keepalive.png
[nano-node-add-initial-peers]: ./images/node/nano-node-add-initial-peers.png
[nano-node-peering-communication]: ./images/node/nano-node-peering-communication.png
[nano-node-send-node-handshake]: ./images/node/nano-node-send-node-id-handshake.png
[nano-node-process-node-handshake]: ./images/node/nano-node-process-node-id-handshake.png
[nano-node-repcrawler-ongoing-crawl]: ./images/node/rep\_crawler/rep-crawler.png







