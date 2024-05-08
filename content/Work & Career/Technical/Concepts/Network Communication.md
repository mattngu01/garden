Establishing a TCP connection:
- client sends SYN (synchronize)
	- SYN contains random sequence number (i.e. 4321) to mark beginning  of sequence of data
- server sends SYN + ACK (synchronize + acknowledgement)
	- initial SYN + 1
- client responds with ACK

Finding where to send info to:
- routing table - know which interface to forward a packet
- client will send request for connection to default gateway (and keeps going to another until it gets to a gateway running BGP)
- BGP (Border Gateway Protocol) routing table contains all public IP's
	- has best route to destination
> This will contain CNN.com’s public IP address and the way to get to it
- Then, you can establish TCP connection


https://syedali.net/2013/08/18/what-happens-when-you-type-in-www-cnn-com-in-your-browser/