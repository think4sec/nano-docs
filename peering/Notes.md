# Peering

Rough draft of the nano peering process. Describes the peer discovery process, state monitoring of peers, and block publishing to peers. 

#[Node Instantiation Class](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1012)

	-> network
	[Call](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1031)
	[Implementation](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L31)

#[Node Start](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1537)
 
	-> network.start();
	-> add_initial_peers(); /* https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L2145 */

	-> ongoing_store_flush(); /* Review for association to peer discovery */

	-> rep_crawler.start();
	-> ongoing_peer_store(); /* https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1699 */

# [Merge Peers](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L535)
