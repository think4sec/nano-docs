# Peering

Rough draft of the nano peering process. Describes the peer discovery process, state monitoring of peers, and block publishing to peers. 

node::node::start

	network.start();
	add_initial_peers(); /* https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L2145 */

	ongoing_store_flush(); /* Review for association to peer discovery */

	rep_crawler.start();
	ongoing_peer_store(); /* https://github.com/nanocurrency/nano-node/blob/b3c7ad295cb31c1e502190dd10bc273db2b821fd/nano/node/node.cpp#L1699 */
	-> network.udp_channels.store_all(*this);
	-> pointer to node
	-> set timeout and lock when calling: node_l->ongoing_peer_store();
	port_mapping.start();
# nano::node::node
	-> online_reps
# nano::rep_crawler::rep_crawler
# nano::network::broadcast_confirm_req 
