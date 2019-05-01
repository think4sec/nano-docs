
![nano-node-peering]

<br/>  

+ [Purpose]()  
+ [Definitions]()  
	* [channel](#channel)  
	* [peer](#peer)  
	* [preconfigured\_peers](#preconfigured-peers)  
	* [raw\_units](#raw-units)  
	* [representative](#representative)   
+ [Peering](#peering)
	* [add\_initial\_peers][add-initial-peers]  
	* [rep\_crawler][rep-crawler]  
+ [Message Types][message-types]  
	* [confirm\_ack][confirm-ack] 
	* [confirm\_req][confirm-req]  
	* [keepalive][keepalive]
	* [node\_id\_handshake][node-id-handshake]
	* [publish][publish] 
* [Cost][cost]
<br/> 
 
### Purpose

This document is intended for anyone looking to understand the nano's peering architecture in an easy to digest format.  


### Definitions

>### <a name="channel"></a>**channel**

>_Channels represent the internal representation of an endpoint and it's port. Although no internal message sub-system exists, the nature of channels is very similar to common messaging systems._

>### <a name="peer"></a>**peer**  
>_Represents the foundation of any distributed network. Peers in the nano network are responsible for replicating transaction blocks to other peers, such that each peer contain the same set of transaction blocks._

>### <a name="preconfigured-peers"></a>**preconfigured_peers**
>_Represents a list of peers that aide in establishing communication with other network participants._

>### <a name="raw-units"></a>**raw units**  
>_Nano contains a total circulating supply is approximately **1.33x10^37** raw units. General measurement is done in **NANO** which is 1x10^29 raw units (**1 NANO**)._

>### <a name="representative"></a>**representative**

>_A representative is a type of peer that contains enough direct/indirect weight to vote on >incoming blocks._

>### <a name="syn-cookies"></a>**syn_cookies**
>
>_Are used to internal identify valid peers a node has established communication with._

<br/>

### <a name="peering"></a>Peering

The peering process starts by identifying and adding any existing peers stored in data store. This internal process is known as **[add\_initial\_peers][add-initial-peers]**. If node contains no existing peers, the internal process named **[rep\_crawler][rep-crawler]** will start the communication process with each peer defined in the **preconfigured\_peers** list. The preconfigured peers are used as seed peers to identify other network participants. 

The default preconfigured peer is **_peers.nano.org_** . The node starts by sending a **[keepalive][keepalive]** message to this peer. After message is sent, node waits idly until peer responds with a keepalive message of its own.

> Keepalive messages are the backbone for nodes to share peer relationships. Each message sent will contain a dynamic list of peers known to node. See [keepalive][keepalive] section for additional information.

By design, a node receiving a keepalive message will identify if peer is already known. In the instance where a peer is unknown, a node will start the **[node\_id\_handshake][node-id-handshake]** process. Once process completes, node will send a keepalive message to new peer.

The image below depicts the general communication flow to preconfigured peers.
 
![nano-node-peering-communication]


>**NOTE:**  
>  
>  preconfigured\_peers configuration entry is only read upon node bootup. Changes to this list will require a process restart to take effect.

Given nodes operate is a geo-distributed environment (internet), peers can become stale (peers no longer online, etc..). In order to avoid staleness of peers, the list of peers is refreshed automatically every **300 seconds**. The refresh process consists of removing all stored peer data from data store, iterating over active communication channels, extracting associated endpoint, and committing them to the data store.

This process is repeated for the life of the node. 

The following sections attempts to describe the core internal processes related to peering.

#### <a name="add-initial-peers"></a>add\_initial\_peers  

This method will validate and verify a node's previous known peers for reuse. Each previous known peer will go through the **node\_id\_handshake** process.

![nano-node-add-initial-peers]  

New nodes will simply have an empty list which would require it to establish communication with the list of preconfigured\_peers.  

#### <a name="rep-crawler"></a>rep\_crawler  

Rep crawler primary purpose is to ensure the node has established enough sufficient weight (_**delegated+peers online weight**_), to properly participate in the network. 

The run-time characteristics of this process is as follows:

| Sufficient Weight | Run-time |
--- | ---
Yes | 7 seconds
No | 3 seconds  

The sufficient weight must be greater than the **online\_weight\_minimum** (_Default: **6x10^37 raw units**_) configuration entry. 

Upon each execution, rep\_crawler attempts to discover weighted peers (_representative_). Initially, last known weighted peers are added to the list of channels, then a random set of peers are selected. The size of random peers selected, range from **15** to **60**. The range is determined by a node's sufficient weight value. 

Once list selection is done, each peer within the list is sent a **[confirm\_req][confirm-req]** message. This message will contain a random block selected from the ledger. Peers are given up to **5 seconds** to respond with a **[confirm\_ack][confirm-ack]** message. If no response is given, block is simply removed from queue. 

![nano-node-repcrawler-ongoing-crawl]  

Internally an observer pattern is used to observed a response _([confirm\_ack][confirm-ack] messages)_ to our request. Each vote response observed, node will check if peer meets expected weight _(**online peers weight / 1000**)_. If weight is met, peer is added to node's internal weighted peer container. Addition of new peer to container will invoke node to submit all known winning blocks _([confirm\_req][confirm-req] message)_ to weighted peer for vote.
 
The next section discusses message types used for peer communication. 

### <a name="message-types"></a>Message Types  

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

#### <a name="confirm-ack"></a>confirm\_ack  

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
 
#### <a name="confirm-req"></a>confirm\_req  

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

#### <a name="keepalive"></a>keepalive  

The purpose of this message type is to share a node's peer list with other network peers.

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

#### <a name="node-id-handshake"></a>node\_id\_handshake  

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
  
#### <a name="publish"></a>publish  

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

### <a name="cost"></a>Cost  

Network cost summary

2 bytes will be added for magic number (network identification)  
8 bytes will be added for header size

Frequency is in seconds  

#### Basic packet cost in terms of size:  

| Message | Size | Frequency | Total  
--- | ---  | --- | --- 
confirm\_ack | 168 bytes -- ?  | &nbsp; | 178 bytes -- ?
confirm\_req | (64 or 216) bytes -- ? | &nbsp; | (74 or 226) bytes -- ?
keepalive | 144 bytes | Min: 3, Max: 7 | 154 bytes
node\_id\_handshake | 96 bytes | &nbsp; | 106 bytes
publish | 216 bytes | &nbsp; | 226 bytes 

#### New weighted peer connected to node:
  
##### Initial Communication  
| Direction | Send Size | Recv Size | Total
--- | --- | --- | ---
send keepalive message | 154 bytes | | 154 bytes
receive node\_id\_handshake message | &nbsp; | 106 bytes | 106 bytes
send node\_id\_handshake message| 106 bytes | &nbsp; | 106 bytes
receive keepalive message | &nbsp; | 154 bytes | 154 byte
&nbsp; | 260 bytes | 260 bytes | **520 bytes**


##### Probe to detected weighted peer
| Direction | Send Size | Recv Size | Total
--- | --- | --- | --- 
send confirm\_req message| 226 bytes | &nbsp; | 226 bytes
receive confirm\_ack message| &nbsp; | 354 bytes | 354 bytes
&nbsp;|226 bytes | 354 bytes | **580 bytes** 

##### Peer added to weighted peer container  

| Direction | Send Size | Recv Size| Total
--- | --- | --- | ---
send confirm\_req message | 226 bytes  | &nbsp; | 226 bytes
receive confirm\_ack message |  | 354 bytes | 356 bytes
&nbsp; | 226 bytes | 354 bytes | **580 bytes** x _winning-blocks_
  

[add-initial-peers]: #add-initial-peers
[keepalive]: #keepalive
[node-id-handshake]: #node-id-handshake  
[rep-crawler]: #rep-crawler
[confirm-ack]: #confirm-ack
[confirm-req]: #confirm-req
[publish]: #publish
[message-types]: #message-types
[cost]: #cost

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







