
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

Section covers the different message types currently used for peer communication. Each message type, is transported with an appropriate message header. The message header will contain the following; **version\_max**, **version\_using**, **version\_min**, **payload message type**, and **extension**.  


**Supported payload message type**  

| Type | Length | Description   
--- | --- | ---  
bulk\_pull | 0 |   
bulk\_pull\_account | 0 |  
bulk\_push | 0 |   
frontier\_req | 0 |    
keepalive | 0 |   

#### confirm\_ack
#### confirm\_req
#### keepalive
#### node\_id\_handshake  

The node\_id\_handshake allows for node's to uniquely identify one another. The message stores the following data:

| Item | Length | Description  
--- | --- | ---
query | 32 bytes | Reference syn_cookie created for peer
response | 64 bytes | Hash containing node's public_id and signature  


###### send

Sending this message type, the node will first allocate a **syn\_cookie** for peer as well signature of message. This signature will be used to verify validity of message. Both **query** and **response** data are populated, before sending message to peer.

![nano-node-send-node-handshake]

###### receive

Receiving this message type, the handler will extract the **query** and **response** data from message payload.

![nano-node-process-node-handshake]

If **existing peer**, verify the following:

- message signature is valid
- handshake is not local
- Verify no existing node id and different port exists, if so remove

Once completed, a new communication channel is created and recorded for this peer, registering its address, port, and node_version.

If **non-existing peer**, assign a syn cookie for this endpoint and start the node handshake communication by sending peer a **node\_id\_handshake** message.

Finally record stats for this handshake occurrence.  

#### publish  

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

New nodes will simply have an empty list which would require it to establish communication with the list of **preconfigured\_peers** to obtain a list of other peers. This internal process known as **ongoing_crawl** will reach out to all preconfigured peers and send a keepalive message. 


[nano-node-peering]: ./images/node/nano-node-peering.png
[nano-node-add-initial-peers]: ./images/node/nano-node-add-initial-peers.png
[nano-node-peering-communication]: ./images/node/nano-node-peering-communication.png
[nano-node-send-node-handshake]: ./images/node/nano-node-send-node-id-handshake.png
[nano-node-process-node-handshake]: ./images/node/nano-node-process-node-id-handshake.png







