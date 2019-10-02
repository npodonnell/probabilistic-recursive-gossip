
# <center>PROBABALISTIC RECURSIVE GOSSIP</center>
<center>A strategy for efficient propogation of state-changes over a peer-to-peer network</center>

 *<center>By N. P. O'Donnell, 2019</center>*


## Abstract

Over the past 20 years, peer-to-peer (P2P) networks have become very popular and continue to be an active area of research. While P2P networks such as Bitcoin, IPFS, FreeNet, and eMule serve different purposes and have different designs, they all share some common requirements: they need to be able to organise themselves without a central authority, discover new nodes joining the network, detect when nodes leave the network, and each participant maintain a sensible number of connections to its peers, such that it may get a reasonable service, making best use of its bandwidth, memory and CPU resources. 

P2P networks also share information or files, and transmit state changes about them. Several designs have been proposed for achieving these goals in a generic way including Pastry, Tapestry, Chord and CAN.

In this paper I present a generic design for peer-to-peer networks which is logarithmically scalable and allows boundless growth of information harboured in the system as a whole, by making each node responsible for a subset of resources, and ensuring that new resources and state-changes pertaining to those resources are routed to the most appropriate nodes in a timely and reliable manner.

First we will discuss a numbering system for tagging each node and resource on the P2P network with a unique ID number such that no central authority is required for dispatching ID numbers, yet no two nodes or resources are likely to share the same ID. We we will then discuss an algorithm wherein each node connects preferably to peers with certain IDs over others, forming a topology where a node can reach any other node on the network with minimal hops. Next, we describe an algorithm which, when run on the nodes, allows this topology to flourish, ensuring quick integration of new nodes into the fabric, and that any breaks in the topology caused by nodes suddenly leaving, are quickly healed. Finally we present an algorithm for propogating state-changes over this topology that is fast, efficient, accurate, fault-tolerant, and scalable.

### 1. Identification of Nodes and Resources

On any kind of network, the participants need a way of identifying themselves, whether it be to a central authority or to each other. On a typical home local-area-network (LAN), there is usually a router running an algorithm called DHCP, which assigns an IP address to any new device joining the network. The router keeps a record of which devices are assigned which IP addresses, and never assigns the same IP address twice, lest packets be mis-routed and arrive at the wrong destination. The router's DHCP service acts as a central authority for IP addresses ensuring this chaotic scenario never occurs. The typical home LAN setup is an example of a centralized network.

Unlike centralized networks, P2P networks have no authority responsible for assigning addresses or IDs, so the participants need a way of choosing an ID for themselves that is highly probable to be unique. Some P2P systems use the IP address of the host they're running on, however this can cause problems if the IP address changes, or the computer is behind a NAT which has other machines also on same P2P network. This could cause outside nodes to think there is only one node on the NAT, when in fact there are several.

One strategy for choosing an ID that's probably unique is to simply use a massive random number as the ID. A number so large that the chances are negligable that any other node has chosen that number. We will use this approach as our method of ID assignment to both nodes and resouces. For both node IDs and resource IDs, we need enough bits in the ID to ensure:

1. We will never run out of IDs
2. We will never randomly choose the same ID twice

We will use the same number of bits for both node IDs and resource IDs because we want certain resources to have an affinity to certain nodes, specifically those nodes who's IDs are numerically close to that of the resources. Finally we would like for both node IDs and resource IDs to be evenly distributed. If we choose a random number modulo <img src="/tex/7801c537bd3e31537f0f7c6c15b473d3.svg?invert_in_darkmode&sanitize=true" align=middle width=95.0370366pt height=27.91243950000002pt/>, these needs are satisfied.

For the examples explained herein we will use a trivial 8 bits for node and resource IDs and pretend that there are never any collisions. For real P2P networks, 8 bits is far too low. 256 or 512 bits is more appropriate. We use the capital letter <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> to denote the number of bits. Therefore, in the PRG system, each node and each resource will have an N-bit ID, giving a range of  <img src="/tex/e51f96ba1c505e6968ed01eb1d7133d6.svg?invert_in_darkmode&sanitize=true" align=middle width=47.171295599999986pt height=27.6567522pt/>. 

For all of the examples, unless otherwise stated, we set <img src="/tex/f67640c49b4decfc49669e14c677015f.svg?invert_in_darkmode&sanitize=true" align=middle width=45.13680929999999pt height=22.465723500000017pt/>.

## 1.1. Modular Distance

Before discussing the network topology, We define the modular distance function. Every node and resource on the network exists on the circumference of an imaginary circle, with <img src="/tex/29632a9bf827ce0200454dd32fc3be82.svg?invert_in_darkmode&sanitize=true" align=middle width=8.219209349999991pt height=21.18721440000001pt/> located at the top of the circle, <img src="/tex/e44df5d151ec50db64d4564130e70ea6.svg?invert_in_darkmode&sanitize=true" align=middle width=32.97044189999999pt height=27.6567522pt/> at the bottom, <img src="/tex/02d7f5ab7b9b7d6402c5fb012e75a432.svg?invert_in_darkmode&sanitize=true" align=middle width=32.97044189999999pt height=27.6567522pt/> at the centre right,  <img src="/tex/b0cef80a9386f09abf42301950b28756.svg?invert_in_darkmode&sanitize=true" align=middle width=61.28084324999998pt height=27.6567522pt/> at the centre left, and all other IDs spaced evenly between them. <img src="/tex/6bd87d9e2f456bcede6b5418622a42a6.svg?invert_in_darkmode&sanitize=true" align=middle width=19.86537134999999pt height=27.6567522pt/> would be located at the same position as zero except it's too high. Any number less than 0 or greater than <img src="/tex/6f0c312b260203e13ba89d79f1cb15d9.svg?invert_in_darkmode&sanitize=true" align=middle width=37.12568144999999pt height=27.6567522pt/> is reduced modulo <img src="/tex/6bd87d9e2f456bcede6b5418622a42a6.svg?invert_in_darkmode&sanitize=true" align=middle width=19.86537134999999pt height=27.6567522pt/> so that it may appear on the circle.

The modular distance, <img src="/tex/9d9b7d3c73b53eee6d7ccdcc892a4950.svg?invert_in_darkmode&sanitize=true" align=middle width=58.81788164999999pt height=22.831056599999986pt/>, between any two IDs, <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/> and <img src="/tex/deceeaf6940a8c7a5a02373728002b0f.svg?invert_in_darkmode&sanitize=true" align=middle width=8.649225749999989pt height=14.15524440000002pt/> on the circle, is the shortest distance along the circumference of the circle, clockwise or anticlockwise, from one ID to the other.

<p align="center"><img src="/tex/bdfa68a1829c52e06880386ae2c901dd.svg?invert_in_darkmode&sanitize=true" align=middle width=470.39956589999997pt height=59.20870889999999pt/></p>

The intuition is that <img src="/tex/9d9b7d3c73b53eee6d7ccdcc892a4950.svg?invert_in_darkmode&sanitize=true" align=middle width=58.81788164999999pt height=22.831056599999986pt/> will be positive if we move a little bit clockwise, or negative if we move a little bit anti-clockwise, however if we move more than half-way around the circle, the value of <img src="/tex/9d9b7d3c73b53eee6d7ccdcc892a4950.svg?invert_in_darkmode&sanitize=true" align=middle width=58.81788164999999pt height=22.831056599999986pt/> "flips around" because the shortest route is now around the other side of the circle and the sign changes.

## 1.1. Logarithmic Modular Distance

Next we define the logarithmic modular distance funtion, <img src="/tex/752352a4a80076e0726421b751b1797e.svg?invert_in_darkmode&sanitize=true" align=middle width=49.4875194pt height=22.831056599999986pt/>. This is nothing but the base-2 logarithm of the value of <img src="/tex/9d9b7d3c73b53eee6d7ccdcc892a4950.svg?invert_in_darkmode&sanitize=true" align=middle width=58.81788164999999pt height=22.831056599999986pt/>. Note that <img src="/tex/752352a4a80076e0726421b751b1797e.svg?invert_in_darkmode&sanitize=true" align=middle width=49.4875194pt height=22.831056599999986pt/> is undefined if <img src="/tex/7defd50098be0a3d3e6d4bf5ca764b65.svg?invert_in_darkmode&sanitize=true" align=middle width=39.96184334999999pt height=14.15524440000002pt/>.

<p align="center"><img src="/tex/b8ee72c2cc22ccdbbe8829fc93734655.svg?invert_in_darkmode&sanitize=true" align=middle width=282.1732188pt height=16.438356pt/></p>

## 1.2. Affinity

Finally we define the affinity function. This function gives us a real number between 0 and 1 which indicates how far away x and y are from each other on the circle:

<p align="center"><img src="/tex/0a341b3022bdb76d737dbaed9abb94d8.svg?invert_in_darkmode&sanitize=true" align=middle width=363.1115136pt height=40.600244849999996pt/></p>

If x and y are the same, their affinity is 1. If they're at opposite ends of the circle, their affinity is 0.

## 2. Network Topology

On some P2P networks, notably the Bitcoin network, a node will connect to as many peers as possible and not express any preference for the kind of peer it connects to. This strategy makes some sense for Bitcoin, as its a very time-sensitive application and nodes want to hear about the latest "block" as soon as possible, and the best way to achieve this is to attempt to connect to everybody. When a new bitcoin block is "mined", it gets bounced around the network chaotically, and every node will soon hear about it, usually more than once.

This is a reliable way of getting the information "out there" about the new block, and works well for Bitcoin since a it has a relatively low number of nodes. New blocks are created rather infrequently (every ten minutes), and there's an upper bound on the size of a block (1Mb). However this simple method of gossip is not satisfactory for systems with a far higher volume of information, especially systems where the overall volume of new information is too great to be handled by any single node.

In addition, it's not always feasible for a node to connect to as many nodes as possible, because there can be penalties in the underlying transport mechanism for each new connction, which causes an O(N) or worse cost in terms of memory, bandwidth and CPU usage. A good example of this is TCP, where each TCP connection needs to send a "keep-alive” on a regular cadence to all it's active connections, even if they're idle. Ideally we'd like to have a predictable upper limit on the number of connections we maintain, which is O(1) in the number of nodes.

At this point we introduce the *Logarithmic Spiderweb* (LS) topology as our network topology of choice for connecting together the nodes on our P2P network. 


|![](./img/unstructured_p2p.png)|![](./img/4_bit_node_fullmesh.png)|![](./img/4_bit_node_3.png)|
|:-|:-|:-|
|*<center>Unstructured P2P network, e.g. Bitcoin</center>*|*<center>Complete LS network with N = 4</center>*|*<center>Same LS network showing node 3's peers</center>*|

The LS topology limits the number of peers for any single node to <img src="/tex/c6d581fab590154cd1835b0234c9bc1a.svg?invert_in_darkmode&sanitize=true" align=middle width=39.65759654999999pt height=22.465723500000017pt/>. Using the PRG self-organising algorithm described in the next section, each node will have approximately <img src="/tex/2ec6e630f199f589a2402fdf3e0289d5.svg?invert_in_darkmode&sanitize=true" align=middle width=8.270567249999992pt height=14.15524440000002pt/> peers where <img src="/tex/2ec6e630f199f589a2402fdf3e0289d5.svg?invert_in_darkmode&sanitize=true" align=middle width=8.270567249999992pt height=14.15524440000002pt/> is given by:

<p align="center"><img src="/tex/43343377f75fd98185d0cf4c52acea4b.svg?invert_in_darkmode&sanitize=true" align=middle width=117.78155565pt height=16.438356pt/></p>

and <img src="/tex/55a049b8f161ae7cfeb0197d75aff967.svg?invert_in_darkmode&sanitize=true" align=middle width=9.86687624999999pt height=14.15524440000002pt/> is the total number of nodes on the network. We can manipulate this formula to get an approximation of the number of nodes on the network given <img src="/tex/2ec6e630f199f589a2402fdf3e0289d5.svg?invert_in_darkmode&sanitize=true" align=middle width=8.270567249999992pt height=14.15524440000002pt/>:

<p align="center"><img src="/tex/e9853dfe3fb70e20bce3902ac364a0cf.svg?invert_in_darkmode&sanitize=true" align=middle width=79.1610336pt height=17.00913885pt/></p>

In a complete and perfect LS arrangement, that is: where every possible node ID is assigned to exactly one node, and each of these nodes is connected to it's "ideal" peers, we can compute the overall number of connections:

<p align="center"><img src="/tex/7e80b702851647d6a9f72109ee65e758.svg?invert_in_darkmode&sanitize=true" align=middle width=156.53208285pt height=14.6502939pt/></p>
<p align="center"><img src="/tex/fe0179a6501443cf1c525b99ddd9bb48.svg?invert_in_darkmode&sanitize=true" align=middle width=209.65768065pt height=14.611878599999999pt/></p>
<p align="center"><img src="/tex/818432ee52a4957bf5300dfd36d45305.svg?invert_in_darkmode&sanitize=true" align=middle width=297.9706587pt height=18.7598829pt/></p>

The ideal peers of a node are the nodes whose IDs are exactly <img src="/tex/529a97fd401d07b4f29ddc1d77f8fd4c.svg?invert_in_darkmode&sanitize=true" align=middle width=81.50228954999999pt height=27.6567522pt/>  away from it where <img src="/tex/c5db0128ae8fb60b9c723c68cfd26f2a.svg?invert_in_darkmode&sanitize=true" align=middle width=106.50768809999998pt height=24.65753399999998pt/>. So if a node's id is <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/>, its ideal peers are:

<p align="center"><img src="/tex/b1d32d131fb1ee9de6bcdd1656c50148.svg?invert_in_darkmode&sanitize=true" align=middle width=710.04645855pt height=18.7598829pt/></p>

We leave out <img src="/tex/0e2682347ee3229716232c0bfcadb82d.svg?invert_in_darkmode&sanitize=true" align=middle width=132.00226049999998pt height=27.6567522pt/> since it always has the same value as <img src="/tex/9cb6027af6b0cfc228a2c69583ed942f.svg?invert_in_darkmode&sanitize=true" align=middle width=143.87424314999998pt height=27.6567522pt/>. Applying this formula to an example, a node with ID 73 would have the following ideal peers:

<p align="center"><img src="/tex/e608a551760989e404e84116d45fea89.svg?invert_in_darkmode&sanitize=true" align=middle width=381.73548435pt height=16.438356pt/></p>

Encouraging nodes to connect to peers with IDs close to these ideal IDs means a node can send a message to any other node on the network, even if not directly connected, with a reasonable number of hops.

## 3. Self-Organizing Algorithm

When a new node joins a P2P network, it needs a way of finding out about other nodes already on the network. Usually the P2P software will ship with a list of hard-coded "seed" nodes, which the joining node can initially connect to. From there, the new node can ask its peers about their peers, then connect to them, and so forth.

In LS networks a similar method is used for bootstrapping, but nodes will prefer to connect to certain peers over others. In particular they favor peers who's IDs are most similar to the node's ideal peers; more precisely the peers which have the smallest logarithmic distance to one of the node's ideal IDs.

To achieve this, a node maintains a series of <img src="/tex/c6d581fab590154cd1835b0234c9bc1a.svg?invert_in_darkmode&sanitize=true" align=middle width=39.65759654999999pt height=22.465723500000017pt/> "slots", each slot corresponding to an ideal ID. The goal is to fill each slot with (i.e. connect to) a peer that has the minimal logarithmic distance to the slot's ideal ID. 

We say that a peer <img src="/tex/df5a289587a2f0247a5b97c1e8ac58ca.svg?invert_in_darkmode&sanitize=true" align=middle width=12.83677559999999pt height=22.465723500000017pt/> "snaps" to an ideal ID, <img src="/tex/45de514e4bd2f5ba36f09fff6b549760.svg?invert_in_darkmode&sanitize=true" align=middle width=11.876949149999989pt height=22.465723500000017pt/> of a node <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> if the positive difference between <img src="/tex/bd44fa69457f01e6537529b60f2096f2.svg?invert_in_darkmode&sanitize=true" align=middle width=96.50233229999999pt height=24.65753399999998pt/> and <img src="/tex/70bdfa4952c7a2fcbd5738303b0554ba.svg?invert_in_darkmode&sanitize=true" align=middle width=96.36440714999999pt height=24.65753399999998pt/> is smaller for <img src="/tex/45de514e4bd2f5ba36f09fff6b549760.svg?invert_in_darkmode&sanitize=true" align=middle width=11.876949149999989pt height=22.465723500000017pt/> than for any of <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/>'s other ideal IDs.

When a node <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> first joins an LS network, it connects to a random peer <img src="/tex/df5a289587a2f0247a5b97c1e8ac58ca.svg?invert_in_darkmode&sanitize=true" align=middle width=12.83677559999999pt height=22.465723500000017pt/>. As soon as the connection is made, <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> and <img src="/tex/df5a289587a2f0247a5b97c1e8ac58ca.svg?invert_in_darkmode&sanitize=true" align=middle width=12.83677559999999pt height=22.465723500000017pt/> exchange their peer lists with each other. When <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> receives <img src="/tex/df5a289587a2f0247a5b97c1e8ac58ca.svg?invert_in_darkmode&sanitize=true" align=middle width=12.83677559999999pt height=22.465723500000017pt/>'s peerlist, <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> iterates through it and attempts to connect to any peer <img src="/tex/ef0de0b48cb187b636ae34b0aea8c1db.svg?invert_in_darkmode&sanitize=true" align=middle width=15.20454704999999pt height=22.465723500000017pt/> who's ID either:

1. Snaps to one of <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/>'s currently empty slots
2. Snaps to one of <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/>'s occupied slots but is closer to the ideal ID than the current occupier

In both these cases, <img src="/tex/f9c4988898e7f532b9f826a75014ed3c.svg?invert_in_darkmode&sanitize=true" align=middle width=14.99998994999999pt height=22.465723500000017pt/> will try to connect to <img src="/tex/ef0de0b48cb187b636ae34b0aea8c1db.svg?invert_in_darkmode&sanitize=true" align=middle width=15.20454704999999pt height=22.465723500000017pt/>  and if successful, close the connection to the previous occupier if there was one, replacing it with <img src="/tex/ef0de0b48cb187b636ae34b0aea8c1db.svg?invert_in_darkmode&sanitize=true" align=middle width=15.20454704999999pt height=22.465723500000017pt/>. Whenever a node successfully connects to a new peer it informs all its other peers of the new connection.
