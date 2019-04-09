# Peering

Rough draft of the nano peering process. Describes the peer discovery process, state monitoring of peers, and block publishing to peers. 

#[Node Instantiation Class][node-instantiation]

	-> network
#### [Call][network-class-instantiation]
####[Implementation][network-class-implementation]

#[Node Start](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1537)
 
#### [network.start()][]
#### [add_initial_peers()][add-initial-peers]

	> ongoing_store_flush(); /* Review for association to peer discovery */
	> rep_crawler.start();
	> ongoing_peer_store(); /* https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1699 */

# [Merge Peers](https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L535)

# Other notes
nano::network::broadcast_confirm_req  
 Fetch/Create list of _representatives_ or _peers_ (if node's connected represenative is less than configuration online_weight_minimum)
	If list is empty or
	If list of _peers_ weight is less than node's configuration _online_weight_minimum_
	Create list of _peers_ with the minimum size of 100 or 2*sqrt(total peers)
 
 Trim list to a maximum of 32 randomly chosen endpoints.
 Call nano::network::broadcast_confirm_req_base

nano::network::broadcast_confirm_req_base
 Limit immediate send to a total of 10 _representatives_ or _peers_ 
 Otherwise, delay broadcast to other remaining _representatives_ or _peers_  upto 10 milli-seconds

nano::node::add_initial_peers
 Create read transaction
 Fetch list of endpoints (review store.peers_begin and store.peers_end method: nano::node::store )
	
 based on nodeconfig.cpp, a total of 8 preconfigured representatives and 1 preconfigured peer set by default
 [Preconfigured Peer/Reps](https://github.com/nanocurrency/nano-node/blob/e3d97cffcbbfe4a71041bf5ac929e54560f81df3/nano/node/nodeconfig.cpp#L48)

# Type Of Peers
 * General Peers
 * Work Peers

## Explain "peering port" option
## Explain "allow_local_peers" option

## Work Peers
nano::block_store
	* parent class to mdb_store (located in lmdb.hpp) 

	nano::store.peers_begin: fetch unique list of entries (key,value)
	nano::store.peers_end: fetch list of all entries

[lmdb.cpp](https://github.com/nanocurrency/nano-node/blob/master/nano/node/lmdb.cpp)
nano::mdb_store::peer_put
nano::mdb_store::peer_del
nano::mdb_store::peer_exists
nano::mdb_store::peer_count
nano::mdb_store::peer_clear
nano::mdb_store::peers_begin
nano::mdb_store::peers_end
[network-class-instantiation]: https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1031
[network-class-implementation]: https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L31
[node-instantiation]: https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1012
[add-initial-peers]: https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L2145
