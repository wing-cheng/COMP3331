# Computer Network

This note contains all the content I learnt from COMP3331 Computer Network and Applications, 2021 T1, at UNSW.

This note is organised in the following structure (from macro to micro):

1. protocol layers
2. internet protocol

# Protocol Layers

Computer network layers are ordered as below:

1. application layer
   - DNS, HTTP, HTTPS, CDN
2. transport layer
   - TCP, UDP
3. network layer:
   - **only** IP: routing protocol
4. link layer
   - ARP
5. physical layer (not important)

## Why must have layers?

Because if we don't have layers, each application has to be re-implemented for every new network technology

Having intermediate layers provides **common** abstraction for various network technology

# Application Layer

## Socket

A door that allows communication between processes in application layer and the transport layer

In order to deliver a message properly, we need both the correct IP address and port number

## Client-Server architecture

Context: *N* clients, *F* file size

**The server** must upload file *N* times, **each client** must download the file copy

Time to distribute *F* to *N* clients:
$$
D_{\text{c-s}} \geq \text{max}\{NF/u_s, F/d_{min}\}
$$

$$
d_{min}: \text{minimum client download speed}
\\
u_s: \text{server upoad speed}
$$



## P2P architecture

A host is both the client and the server, but it needs at least one peer online to start the process

e.g. BitTorrent, KanKan, Skype, BitCoin (Cryptocurrency)

**The server** must upload at one copy of the file, **each client** must download one file copy, so **all clients** downloads *NF* bits

Time distribute *F* to *N* clients:
$$
D_{\text{P2P}} \geq \text{max}\{F/u_s, F/d_{\text{min}}, NF/(u_s + \sum_{i=1}^Nu_i)\}
\\
u_i: \text{client upload speed}
$$

#### P2P is lot more faster than Client-Server in distribution time

### Pros

- peers provide service to each other, beneficial to each other (I share something to you, you share something else to me as the return)
- saleable: a new peer joining provides more service capacity
- parallelism, no contention
- geographic distribution

###  Cons

- decentralised: lack of control
- distributed algorithms are complex

## DNS

- port 53

### Context

Every internet host has a 32-bits IP address and a name that is readable for human (e.g. "www.google.com"). We need a mapping between IP address and the name.

- **distributed database system**  that is implemented in hierarchy of name servers
- **application-layer protocol** hosts communicate with name servers to translate between names and IP addresses 

### Why UDP not TCP?

1. DNS lookups need to be quick, UDP is quick and has low overhead. TCP is slower as it requires 3-way handshake
2. DNS requests are small and can be fitted into a single UDP segment
3. Though DNS needs to be reliable and UDP is not reliable, we can always implement reliability on top of application layer to account for the unreliability of UDP

### Why not centralise DNS?

- single point of failure
- traffic volume
- distant centralized DB
- maintenance
- does not scale

### Server Hierarchy

- Top-level: root servers
  - every server knows the root server
  - root server knows all TLDs
- Next-level: top-level domain (TLD) servers
  - .com, .edu, .au, .uk, .cn
- Bottom-level: authoritative DNS servers
  - stores name-to-address mapping
  - maintained by corresponding administrative authority
- Local DNS name server
  - NOT belong to hierarchy
  - each ISP has one
  - is the default name server
  - DNS queries are sent to local DNS, local DNS will ask the hierarchy

![server hierarchy](https://github.com/wing-cheng/COMP3331/blob/main/img/typora-icon.png)

### Example

**A host wants to open a webpage when all of its caches are empty. The 1st frame that will be transmitted from this host will belong to which protocol? Assume that the host has already acquired an IP address.**

DNS

The host needs to map the alias web address to an IP address

### Reverse DNS

- maps IP addresses to names

#### Where do we use reverse DNS?

- trouble shooting (*traceroute*, *ping*)
- in SMTP servers for validating IP addresses of organizaions
- in load balancing servers/content distribution to determine location of requesters

## CDN (content distribution networks)

### Context

##### **Challenges**

- video traffic is the major consumption of internet bandwidth
  - Netflix, 37% of downstream residential ISP traffic
  - YouTube, 16%
- scale, how to reach ~2B per user?
- different users have different capabilities (wired VS mobile)

##### **Solution**

- distributed, application-layer infrastructure

### Video Solution

**A video is just a sequence of images displayed at a constant rate**

- use redundancy **within** and **between** images to reduce #bits to encode the image
  - CBR (constant bit rate)
    - video encoding rate fixed
  - VBR (variable bit rate)
    - video encoding rate varies as the amount of spatial and temporal coding varies
    - not much motions, low VBR
    - much motions, high VBR

### DASH (Dynamic, adaptive streaming over HTTP)

**Server**

- divides a video file in multiple chunks
- each chunk maybe encoded at different rate
- **manifest file**: provide different URLs for different chunks

**Clients**

- periodically measures the server-client bandwidth
- consult manifest, request for 1 chunk at a time
  - choose the max encoding rate based on available bandwidth
  - based on current bandwidth, choose different encoding rate at different point of time

### Why distributed?

#### Challenge

How to stream out contents to hundreds of thousands of simultaneous users?

#### Solution

Store / server multiple copies of contents at multiple geographical locations

1. push CDN servers deep into many access networks
   - expensive but better user experience
2. small number (10's) of large clusters near access network
   - cheap but not so good user experience

## HTTP (hyper text transfer protocol)

- needs at least 1 server and client(s)

### Statelessness

Server does **not** maintain any info about the client

### Request and Response Messages

Messages can be sent using **TCP** or **UDP** protocol (transport layer)

### HTTP versions

- HTTP/1.0
  - GET: request page
  - POST: upload user response
  - HEAD: ask server to leave requested object out of response
- HTTP/1.1
  - GET, POST, HEAD
  - PUT: upload files(s) in entity body to path specified in URL field(s)
  - DELETE: delete file(s) specified in URL field(s)
  - TRACE, OPTIONS, CONNECT, PATCH: for persistent connection

### Performance

Decrease PLT (page load time)

- smaller content size
  - compression
  - smaller images
- improve HTTP
  - persistent connection
  - pipe-lining
  - caching
- smaller travel distance
  - CDNs

#### Non-persistent HTTP

response time:

- 1 RTT for initiate TCP connection
- 1 RTT for HTTP request and response to return

total = 2 RTT + file transmission time

##### Concurrent Request & Response

- multiple connections in parallel
  - but still 1 TCP connection for each file
- no maintenance of the order in which the files are received

#### Persistent HTTP

- 1 TCP connection for multiple files

##### With pipe-lining

- send 1 file at once
  - so 1 RTT for each file

##### Without pipe-lining

- send all files at once
  - so 1 RTT for all files

### HTTP/1.0

- non-persistent
- 1 TCP connection for each file to transmit
  - so, 5 files takes at least 5 * 2 RTT to transmit

### HTTP/1.1

- persistent TCP connection with pipe-lining

## SMTP

### **Components**

- user agents
- mail servers
- mail transfer protocol: SMTP

#### Mail servers

- **mail box** storing incoming mails
- **message queue** of outgoing mail
- **SMTP** protocol between mail servers

### Scenario: *A* sends a mail to *B*

![](/home/kiki/Pictures/Screenshot from 2021-05-21 16-00-53.png)

# Transport Layer

### Multiplexing/Demultiplexing

- multiplexing at **server**
  - handles data from multiple sockets
  - adds transport header

- demultiplexing at **clients**
  - use the header to deliver segments to the correct socket

## UDP

- only use one socket for UDP segments
- one socket for all requests

### Applications

- latency critical
  - gaming
  - voice / video chat
  - quick request / response (DNS)

## TCP

- multiple sockets listen to the same port (all socket have the same port number)
- one socket for one request
- one welcome socket

### Features of Reliable Data Transfer

- checksum
- sequence number (byte offset)
- cumulative acknowledgements
  - handles ACK loss
- receiver buffers out-of-sequence packets
- sender maintains a retransmission timer + retransmit at timeout

### Congestion Control

#### Large Retransmission Timeout

When a TCP sender does not have any initial estimate of RTT (round-trip time), it sets a very large RTT such that it does not do unnecessary or excessive retransmission and thus does not exhaust the network, which is resulted from using a RTO (retransmission timeout) that is smaller than the actual RTT.

#### Congestion Window (CWND)

- #bytes sent without overflowing the **routers**

#### Advertised / Receiver Window (RWND)

- #bytes sent without overflowing the **receiver**

#### Sender Window

- min {CWND, RWND}

#### Sender detects congestion

- Duplicate ACKs
  - some segments are loss, some are delivered
- Timeout
  - not enough dup ACKS
  - several losses

#### Sender adjust sending rate

Slow Start

- new ACK
  - if CWND < ssthresh
    - CWND += 1
  - if CWND >= ssthresh
    - CWND += 1 / CWND
- dup ACKs
  - if dupCount = 3 (fast retransmit)
    - ssthresh /= 2
    - CWND /= 2
- timeout
  - ssthresh /= 2
  - CWND /= 2

| TCP flavor | new ACK                                             | dup ACKs  | timeout |
| ---------- | --------------------------------------------------- | --------- | ------- |
| Tahoe      |                                                     | CWND= 1   | CWND= 1 |
| Reno       | if CWND < ssh, CWND *= 2; if CWND >= ssh, CWND += 1 | CWND /= 2 | CWND= 1 |

### Example

**Consider that only a single TCP Reno connection uses 1 10Mbps link which does not buffer any data. Suppose that this link is the only congested link between the sending and receiving hosts. Assume that the TCP sender has a huge file to send to the receiver and the receiver's receive buffer is much larger than the congestion window. We also make the following assumption:**

**each TCP segment is 1500 bytes**

**the 2 way propagation delay of this connection is 150 msec**

**this TCP connection is always in congestion avoidance phrase (ignore slow start)**

1. What is the maximum window size?

   Let W be the max window size measured in segments.

   W * MSS / RTT = 10Mbps, as packets will be dropped if the max sending rate exceeds link capacity.

   Thus, we have W * 1500 / 0.15 = 10^6, W ~= 125 segments

2. What is the average window size (in segment) of this TCP connection?

   As congestion window size varies from W / 2 to W, then the avg window size is 0.75 * W ~= 94 segments.

3. How long would it take for this TCP connection to reach its maximum window again after recovering from a packet loss?

   We know that TCP reno will halve the window size whenever there is a packet loss.

   We need to receive 125 / 2 = 63 ACKs to recover the max window size, 1 ACK is received every RTT, hence the time taken is 125 / 2 * 0.15 = 9.375 sec

### Example

**Assume that the SendBase for a TCP Reno sender is currently 4000. The TCP sender has sent 4 TCP segments with sequence number 4000, 4500, 5500 and 7000. The sender then receives a segment with an acknowledgement number 7500 and a receive window 6000. The congestion window, CongWin, is set to 10,000 bytes after this ACK is processed. Assume that this ACK is processed and no furthur ACKs are received.**

1. What is the value of SendBase?

   7500

   The sender has received the ACK with ACK number 7500, hence the sender should change the SendBase to 7500 as well

2. How many bytes in total are sent in the 4 segments?

   3500

   The 1st segment has 500 bytes, the 2nd has 1000 bytes, the 3rd has 1500 and the last has 500 bytes. 500 + 1000 + 1500 + 500 = 3500.

   7500 - 4000 = 3500.

3. What is the last byte (number) that the TCP sender can send with certainty that the receiver's buffer will not overflow? Assume that the sender always has data to send.

   Window size = min{CongWin, RecWin} = min{10,000, 6000} = 6000

   Since current SendBase is 7500, hence we have the last byte being 6000 + 7500 - 1 = 13499

4. Now assume that the sender receives 3 more TCP segments, such that all 3 segments hace TCP acknowledgement number 7500. Assume that all 3 ACKs are processed and no furthur ACKs are received.

   Since TCP reno halves the congestion window size for 3 duplicated ACKs, now, CongWin = 5000



# Network Layer

### 2-key functions of Network layer

1. forwarding

   - process of getting through single interchange
   - move packets from router input to appropriate router output

2. routing

   - process of planning trip from source to destination

   - determine route taken by packets from source to destination

## IP (Internet Protocol)

### Fragmentation

#### Example

**An IP fragment with 600 bytes in the data payload and an offset field set at 900, is furthur fragmented into 2 fragments with data payload of size 400 and 200. The offset fields in these 2 fragments are**

900 and 950

the 1st fragment has 900 as offset

the 2nd fragment has 200 bytes of data, 400 / 8 = 50, 900 + 50 = 950. So it has 950 as the offset.

### IP Address

#### Example

**A small campus is assigned a large address block 12.1.0.0/17, but is only using a portion of these addresses (in 12.1.1.0/24) to number its computers. The campus ueses a single Internet Service Provider (ISP) to reach the rest of the internet. The pic below shows the forwarding tables on the ISP's router (on the left) and the campus edge (on the right).**

![](/home/kiki/cs3331/OneDrive-2021-06-04/截圖 2021-05-05 下午9.11.06.png)

**E.g. the ISP forwards all packets with destination addresses in 12.1.0.0/17 to link #2 toward the campus edge router. Both routers include a default forwarding entry (i.e. 0.0.0.0/0) that can match any destination IP address.**

1. How many IP addresses does the campus "own" in its 12.1.0.0/17 block?

   The total number of bits in an IP address is 32. 32 - 17 = 15 bits can be used by campus. Hence, 2^15 addresses owned by this campus.

2. What are the smallest and largest address IP address that the campus "owns"? Does these addresses have special meaning and if so what do they signifity?

   Smallest: 12.1.0.0, this means the network address (refers to the entire network)

   Largest: 12.1.127.255, this is the boardcast address (refers to every host within the network)

3. Suppose the ISP router receives a packet from the internet with destination IP address 12.1.1.1. What path does this packet follow (indicate the path using link number from the figure above)

   1 -> 2 -> 3 -> 4

4. Suppose the ISP router receives a packet from the internet with destination IP address 12.1.20.1? What path does this pakcet follow (indicate the path using link numbers from above figure)? What is the ultimate outcome for this packet?

   1 -> 2 -> 3 -> 2 -> 3 -> 2 -> 3 ... It keeps looping between link 2 and 3. The router will discard this packet until its TTL goes to 0.

#### Example

**Assume that the forwarding table of a router is as follows:**

![](/home/kiki/cs3331/OneDrive-2021-06-04/截圖 2021-05-05 下午9.21.24.png)

1. Which interface would a datagram with destination IP address 128.96.171.92 be forwarded to?

   0

2. Which interface would a datagram with destination IP address 128.96.163.15 forwarded to?

   4

3. Which interface would a datagram with destination IP address 128.96.165.121 be forwarded to?

   3

## Routing Algorithms

### Example

Consider the following network topology comparising 6 routers labelled A through F. The link costs (in both direction) are shown

![](/home/kiki/cs3331/OneDrive-2021-06-04/截圖 2021-05-05 下午10.42.47.png)

1. Show the operation of Dijkstra's link-state algorithm to compute routes from node A to all destination.

   | Step | N'   | D(B), p(B) | D(C), p(C) | D(D), p(D) | D(E), p(E) | D(F), p(F) |
   | ---- | ---- | ---------- | ---------- | ---------- | ---------- | ---------- |
   | 0    | A    | 20         | 2          | 2          | 2          | -          |
   | 1    | C    | 20         | 2          | 2          | 2          | -          |
   | 2    | D    | 5          | 2          | 2          | 2          | 5          |
   | 3    | E    | 3          | 2          | 2          | 2          | 5          |

2. Based on your execution of Dijkstra's algorithm in the abve question, complete the forwarding table for nodes.

   ![](/home/kiki/cs3331/OneDrive-2021-06-04/截圖 2021-05-05 下午11.02.59.png)

   | Destination | Outgoing Link |
   | ----------- | ------------- |
   | B           | (A,E)         |
   | C           | (A,C)         |
   | D           | (A,D)         |
   | E           | (A,E)         |
   | F           | (A,D)         |

3. Show the distance table that would be computed by the distance vector algorithm in A once the distance vector algorithm has finished executing.

   | Destination | via B     | C        | D       | E        |
   | ----------- | --------- | -------- | ------- | -------- |
   | B           | 20        | 7(ACAEB) | 5(ABD)  | 3(AEB)   |
   | C           | 25(ABEAC) | 2        | 6(ADC)  | 6(AEAC)  |
   | D           | 23(ABD)   | 6(ACAD)  | 2       | 6(AEBD)  |
   | E           | 21(ABE)   | 6(ACAE)  | 6(ADBE) | 2        |
   | F           | 26(ABDF)  | 9(ACADF) | 5(ADF)  | 9(AEBDF) |

4. Consider 3 instances of link failure: (i) link A-B fails; (ii) link A-C fails; (iii) link A-E fails. In each of these cases, all links except the one mentioned are still active. In which of the 3 instances of link failure will the count-to-infinity problem occur?

   - Link A-B fails would not effect the routing table since this link is not part of the shortest path topology. So, we do not need to do anything to account for that.

   - Link A-C; since this link is part of the shortest path from A to C, A will realize that its shortest path to C fails, it will try to find a new path that routes to C. It checks its routing table and find that the shortest path available is to route through D (cost 6), without realizing that this path also uses link A-C. A makes D the next hop neighbour for its route to C, and advertise the new cost 6 for its shotest path to C. D then updates its cost to 8 since the cost to 8 since the cost to get from D to A is 2 and the adveritsed cost from A to C is 6. Thus, A and D 'count to infinity' by updating costs by 2 each time. Once the cost through A becomes greater than or equal to 20, D chooses the link D-C as the shortest path and advertised 20 to A. A updates its cost to 22, and the count stops.
   - link A-E fails; A will find the next available shortest path to E, which is the path routing through D, given that link A-C fails, with cost 6 (ADBE), without realizing that D's next hop is A. A makes D the next hop neighbour for its route to E and advertises the new cost 6 for its route to E. D then choose chooses the its shortest path to D to be D->B->E. Count to infinity should not happen in this case.

### Example

**Consider the network shown in the figure below. Assume that al switch tables are empty at the start.**

![](/home/kiki/cs3331/OneDrive-2021-06-04/image-20210506131720433.png)

1. Assume that host A sends a frame to host F. Indicate all links in the network that this frame is transmitted on and explain why.

   1. the frame is first sent on link A -> S1. When it arrives at S1, since the switch table is empty, the frame will be boardcasted by S1 to all links S1 -> B, S1 -> C and S1 -> S4.

   2. when the frame reaches S4, since the switch table is empty, S4 will boardcast the frame to link S4 -> S2.

   3. when the frame arrives at S2, since the switch table is empty, S2 will boardcast the frame to all links S2 -> D, S2 -> E, S2->F.

2. Assume that host F now sends a frame to host A. Indicate all links in the network that this frame is transmitted on and explain why.

   1. the frame will be first transmitted on F -> S2. When the frame reaches S2, it would know that host A can be reaches through link S2 -> S4.
   2. as the frame reaches S4, S4 would know that host A is accessed through link S4 -> S1, so it forwards the frame to S1.
   3. Lastly when the frame is at S1, S1 forwards the frame to A through link S1 -> A.

### Example

**Consider the network shown below**

![](/home/kiki/cs3331/OneDrive-2021-06-04/image-20210506133149397.png)

1. Assign an IP address to the leftmost interface of the router, given that the network part of the IP address is 24 bits (i.e. the network is using a /24 addresses block).

   Any address with 111.111.111.\*\*\*\* is fine, except for (111.111.111.111, 111.111.111.0, 111.111.111.112, 111.111.111.255)

2. Suppose that A wants to send an IP datagram to B and knows B's IP address. Must A also know B's MAC address to send the datagram to B? If so, how does A get this information? If not, explain why not.

   Yes, since A and B are the same netowork, A will have to know B's MAC address for it to send a datagram to B. A determines that B is in the same network by applying the subnet mask of the network to B's IP address (result will be the same network address as A's). A will obtain B's MAC address through the ARP protocol.

3. Suppose that A wants to send an IP datagram to C and knows C's IP address. Must A also know C's MAC address to send the datagram to C? If so, how does A get this info? If not, explain why not.

   No, A does not need to know C's MAC address in order to send a datagram to C. A forwards the frame to router, the router will then de-capsulate the datagram and then re-capsulate the datagram in a frame to be sent to the right subnet. R will run ARP protocol to determine C's MAC address, A will not.

4. Suppose that R has a datagram (that was originally sent by A) to send to C. What are the MAC addresses on the frame that is sent from R to C? What are the IP addresses in the IP datagram encapsulated within this frame?

   source IP 111.111.111.111

   destination IP: 222.222.222.222

   source MAC: 1A-23-F9-CD-06-98 (right interface of R)

   destination MAC: 49-BD-D2-C7-56-2A (node C)

### Example

**Suppose 2 hosts have a long-lived TCP session over a path with a 100 msec round-trip time (RTT). Then, a link fails, causing the traffic to flow over a longer path with a 500 msec RTT. This scenario is depicted in the figure below. The original path is the straight path at the bottom. The new path is at the top**

![](/home/kiki/cs3331/OneDrive-2021-06-04/image-20210506140747288.png)

1. Suppose the router on the left recognises the failure immediately and starts forwarding data packets over the new path, without losing any packets. Assume that the router on the right recognises the failure immediately and starts directing ACKs over the new path, without losing any ACK packets. Why might the TCP sender retransmit some of the data packets anyway?

   TCP bases its retransimission timeout (RTO) on an estimate of the RTT between the sending and receiving hosts. In this example, the RTT is 100 msec before the failure. As this connection has been active for some time, the sender's RTT estimate should be pretty accurate. The RTO is typically twice the RTT estimate. When the failure occurs, the increases in the actual RTT implies that the ACK packets will not arrive before the RTO expires. This cases the sender to presume the data packets have been lost. leading to retransmissions, despite that no packet is actually lost.

2. Suppose instead that the routers do not switch to the new paths all that quickly, and the data packets (and ACK packets) in flight are all lost. What new congestion window size does the TCP sender use? Explain your answer.

   The TCP sender's congestion window size depends on how packet loss is detected. If it is duplicate ACKs, the window size will be halved; if it is retransmission timeout, the window size will be set to 1. But since in this case, ACKs were all lost, the sender will not even receive duplicate ACKs, it will detect timeout and thus the sender will set the congestion window size to be.



