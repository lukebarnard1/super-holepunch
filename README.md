# Super Holepunch
Overcoming problems faced behind NAT-enabled routers using a combination of UDP and TCP.

[Here](https://en.wikipedia.org/wiki/TCP_hole_punching), you can see the basics of TCP hole punching. In brief, the two peers, both potentially behind a NAT-enabled gateway must do a TCP simultaneous open. When they do this, both NAT devices register the destination IP address in a routing table. When the initial SYN packets arrive at the opposite NAT devices, the router allows them to pass through to the other peer.

## Problems
There are a few problems with TCP holepunching:
  - Predicting which port the other NAT device is using for incoming connections. When the NAT devices translate the origin IP address of outgoing traffic, they may also choose a different port number because two peers on the private network side may be using TCP connections originating from the same port. So predicting which port the NAT will use to listen for the incoming SYN is non-trivial.
  - The connection has to be synchronous. The TCP state machine allows for a connection to be established following a simultaneous open. Making sure both peers start the connection at the same time is also non-trivial, but not impossible. The idea is that maybe an initial [UDP holepunch](https://en.wikipedia.org/wiki/UDP_hole_punching) can be used to arrange for a future time at which to start the TCP holepunch.

## Solution
  - With an initial UDP holepunch, a time in the future is agreed upon by both peers. 
  - In order to do a UDP holepunch, a server known by both peers must be used. The details can be found [here](https://en.wikipedia.org/wiki/UDP_hole_punching#Flow), but basically this server stores the remote port number of each incoming UDP packet until a packet is received signalling the request of another peer's port number. The server may have to wait indefinetely before receiving the information requested and it may never arrive.
  - Once the server has sent back a UDP port number to a peer, the peer can now send another two UDP packets to the other peer that it wishes to holepunch to. This should now allow the two peers to communicate without the NAT devices dropping packets.
  - The future time is agreed upon over UDP. 
  - The peers must now predict which ports the opposite NAT devices will use. Predicting how the NAT devices will assign a port number to the outgoing TCP connection will be the biggest problem because it is not a standardised behaviour. This prediction can be done through using a server known by both peers (not necessarily the same server). Both peers begin to make two TCP connections to the server and the difference between remote ports of each SYN can be used by the server to predict future external ports of the NAT device's TCP connections. This prediction is sent back to each peer.
  - The peers communicate over UDP their predicted NAT-device TCP ports.
  - At the time specified (<1s in the future), the two peers both open TCP connections with the predicted ports and IP addresses of the target NAT devices.

