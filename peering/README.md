
![nano-node-peering]

<br/>  

+ [Purpose]()  
+ [Definitions]()  
	* [channel]()  
	* [peer]()  
	* [preconfigured\_peers]()  
	* [raw\_units]()  
	* [representative]()   
+ [Peering](/peering/?#peering)
	* [add\_initial\_peers](/peering/?#add_initial_peers)  
	* [rep\_crawler](/peering/?#rep_crawler)  
+ [Message Types](/peering/?#message_types)  
	* [confirm\_ack](/peering/?#confirm_ack)  
	* [confirm\_req](/peering/?#confirm_req)  
	* [keepalive](/peering/?#keepalive)
	* [node\_id\_handshake](/peering/?#node_id_handshake)  
	* [publish](/peering/?#publish) 
* [Cost](/peering/?#cost)
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
<div id="peering"/></div>

### Peering

The peering process starts by identifying and adding any existing peers stored in data store. This internal process is known as **[add\_initial\_peers](/peering/?#add\_initial\_peers)**. If node contains no existing peers, the internal process **[rep\_crawler](/peering/?#rep\_crawler)** will start the process of communication with **preconfigured\_peers**. These preconfigured\_peers are used as seed peers to identify other network participants. 

The default **preconfigured\_peer** is **_peers.nano.org_** . Communication starts by node sending a **keepalive** message (_See [keepalive](/peering/?#keepalive) message type_) to the list of preconfigured peers. Keepalive messages are the backbone to a node building a list of network participants. 

By design, a node receiving a keepalive message will identify if peer is already known. In cases where peer is unknown, node starts up the **[node\_id\_handshake](/peering/?#node_id_handshake)** process with new peer. Once process completes, node will respond with it's own keepalive message. 

The image below depicts the general communication flow to preconfigured peers.
 
![nano-node-peering-communication]


>**NOTE:**  
>  
>  preconfigured\_peers configuration entry is only read upon node bootup. Changes to this list will require a process restart to take effect.

Given nodes operate is a geo-distributed environment (internet), peers can become stale (peers no longer online, etc..). In order to avoid staleness of peers, the list of peers is refreshed automatically every **300 seconds**. The refresh process consists of removing all stored peer data from data store, iterating over active communication channels, extracting associated endpoint, and committing them to data store.

This process is repeated for the life of the node. The following contains a review of these core internal processes.  

<div id="add_initial_peers"></div>  

#### add\_initial\_peers  

This method will validate and verify a node's previous known peers for reuse. Each previous known peer will go through the **node\_id\_handshake** process.

![nano-node-add-initial-peers]  

New nodes will simply have an empty list which would require it to establish communication with the list of **preconfigured\_peers**.  

<div id="rep_crawler"></div>  

#### rep\_crawler  

Rep\_crawler will run every **7 seconds** if node contains sufficient weight (_**delegated+peers online weight**_). Otherwise, run-time is every **3 seconds** until node has communicated with enough peers to satisfy sufficient weight requirement. The sufficient weight must be greater than the **online\_weight\_minimum** (_Default: **6x10^37 raw units**_) configuration entry. 

Upon each execution, rep\_crawler attempts to build a list of peers to poll. It first adds the last known weighted peers to list, prior to adding a set of random peers. The size of random peers to poll range from **15** to **60**. This range is determined by a node's sufficient weight value. Once list is complete, each peer is polled and sent a **confirm\_req** message. This message will contain a random block selected from the ledger. Peers are given up to **5 seconds** to respond with a **confirm\_ack** message. If no response is given, block is simply removed from queue. 

![nano-node-repcrawler-ongoing-crawl]  

<div id="message_types"></div>  

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

<div id="confirm_ack"></div>  

#### confirm\_ack  

The purpose of this message type is to send **votes** to other peers for specific blocks.

The message contains the following members:  

| Member | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
vote | &nbsp; | A nano::vote object  

A vote object contains the following members:

| Member | Size | Description
--- | --- | ---  
sequence | 8 bytes | Vote round sequence number
blocks | (Min: 32 bytes, Max: 216 bytes) x _n_ | Blocks or Block Hashes vote intended for
account | 32 bytes | Account that is voting  
signature | 64 bytes | Signature of **sequence** + **blocks**
hash_prefix | 32 bytes | &nbsp;

###### send  

###### receive  

![nano-node-confirm-ack-recv]  

<div id="confirm_req"></div>  

#### confirm\_req  

The purpose of this message type is to request **votes** from other peers for specific blocks. 

The message contains the following members:  

| Member | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
block | 216 bytes | See block definition in [publish]() section
roots\_hashes | 64*_n_ | A set of _n_ \<hash,root\> pairs. <br/>**hash** = digest of hashables in block. <br/>**root** = previous block or account number for open blocks

###### send  

###### receive

The message will either contain the **block** or **roots_hashes** member populated. 

![nano-node-confirm-req-recv-overview]

**Contain Block**

![nano-node-confirm-req-recv-contain-block]  

**Contain Root Hashes**  

![nano-node-confirm-req-recv-contain-hashes]

<div id="keepalive"></div>  

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

<div id="node_id_handshake"></div>  

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

<div id="publish"></div>  

#### publish  

This message type is used to **publish** blocks to network.

This message contains the following members:  

| Members | Size | Description
--- | --- | ---  
header | 8 bytes | Message header
block | 216 bytes| Block transaction

A block contains the following members:

| Members | Size | Description
--- | --- | ---  
hashables | **account** = 32 bytes<br/>**previous** = 32 bytes<br/>**representative** = 32 bytes<br/>**balance** = 16 bytes<br/>**link** = 32 bytes<br/> | Represents block's associated account, <br/>previous hash in chain, account representative, balance, link field,<br/> size = <br/>(summation for sizeof account+previous+representative+balance+link).
signature | 64 bytes | Represents signature of block
work | 8 bytes | Represents compute work for block


Total block size is: **216** bytes  

Hashables:

| Members | Size | Description
--- | --- | ---
account | 32 bytes | Source account for block transaction
previous | 32 bytes | Previous block transaction for account
representative | 32 bytes | Account handling votes on source account behalf
balance | 16 bytes | Balance of source account
link | 32 bytes | Multi-usage: (when)<br/>**sending** = destination account<br/>**receiving** = source block hash from senders account

###### send  
###### receive 

![nano-node-publish-recv]  

<div id="cost"></div>  

### Cost  

Network cost summary

2 bytes will be added for magic number (network identification)  
8 bytes will be added for header size

Frequency is in seconds  

| Message | Size | Frequency | Total  
--- | ---  | --- | --- 
confirm\_ack | 168 bytes -- ?  | &nbsp; | 178 bytes -- ?
confirm\_req | (64 or 216) bytes -- ? | &nbsp; | (74 or 226) bytes -- ?
keepalive | 144 bytes | Min: 3, Max: 7 | 154 bytes
node\_id\_handshake | 96 bytes | &nbsp; | 106 bytes
publish | 216 bytes | &nbsp; | 226 bytes 


[nano-node-confirm-ack-recv]: ../images/node/nano-node-confirm-ack-recv.png
[nano-node-confirm-req-recv-contain-block]: ../images/node/nano-node-confirm-req-recv-contain-block.png
[nano-node-confirm-req-recv-contain-hashes]: ../images/node/nano-node-confirm-req-recv-contain-hashes.png
[nano-node-confirm-req-recv-overview]: ../images/node/nano-node-confirm-req-recv-overview.png
[nano-node-publish-recv]: ../images/node/nano-node-publish-recv.png
[nano-node-peering]: ../images/node/nano-node-peering.png
[nano-node-send-keepalive]: ../images/node/nano-node-send-keepalive.png
[nano-node-process-keepalive]: ../images/node/nano-node-process-keepalive.png
[nano-node-add-initial-peers]: ../images/node/nano-node-add-initial-peers.png
[nano-node-peering-communication]: ../images/node/nano-node-peering-communication.png
[nano-node-send-node-handshake]: ../images/node/nano-node-send-node-id-handshake.png
[nano-node-process-node-handshake]: ../images/node/nano-node-process-node-id-handshake.png
[nano-node-repcrawler-ongoing-crawl]: ../images/node/rep\_crawler/rep-crawler.png







