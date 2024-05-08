https://aws.amazon.com/what-is/cidr/

> Classless Inter-Domain Routing (CIDR) is an IP address allocation method that improves data routing efficiency on the internet. Every machine, server, and end-user device that connects to the internet has a unique number, called an IP address, associated with it. Devices find and communicate with one another by using these IP addresses. Organizations use CIDR to allocate IP addresses flexibly and efficiently in their networks.



An IP address has two parts:

- The _network address_ is a series of numerical digits pointing to the network's unique identifier 
- The _host address_ is a series of numbers indicating the host or individual device identifier on the network

Classful addresses would have prefix bits (think `192.168.1` for local networks) to indicate network unique id, then append host address (`(192.168.1).100` -> 100 = host addr)


> Classless or Classless Inter-Domain Routing (CIDR) addresses use variable length subnet masking (VLSM) to alter the ratio between the network and host address bits in an IP address. A subnet mask is a set of identifiers that returns the network address’s value from the IP address by turning the host address into zeroes. 

A VLSM (Variable Length Subtnet Mask) sequence breaks down an IP address space into subnets of various sizes. Each subnet can have a flexible host count and a limited number of IP addresses. A CIDR IP address appends a suffix value (`/24`) stating the number of network address prefix bits to a normal IP address.

For example, `192.0.2.0/24` is an IPv4 CIDR address where the first 24 bits, or `192.0.2`, is the network address.