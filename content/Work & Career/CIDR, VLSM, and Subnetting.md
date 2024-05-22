## Definition

https://aws.amazon.com/what-is/cidr/

CIDR refers to the set of standards for allocating IP addresses uniquely.

An IP address has two parts:

- The _network address_ is a series of numerical digits pointing to the network's unique identifier 
- The _host address_ is a series of numbers indicating the host or individual device identifier on the network

Class / network addresses have prefix bits (think `192.168.1` for local networks) to indicate network unique id, then append host address (`(192.168.1).100` -> 100 = host addr). 


> Classless or Classless Inter-Domain Routing (CIDR) addresses use **V**ariable **L**ength **S**ubnet **M**asking (VLSM) to alter the ratio between the network and host address bits in an IP address. A subnet mask is a set of identifiers that returns the network address’s value from the IP address by turning the host address into zeroes. 

A VLSM sequence breaks down an IP address space into subnets of various sizes. Each subnet can have a limited # of IPs, which can be tied to a host. 

A CIDR IP address appends a suffix value (`/24`) or subnet mask stating the number of network address prefix bits to a normal IP address.

For example, `192.0.2.0/24` is an IPv4 CIDR address where the first 24 bits, or `192.0.2`, is the network address. 

## Example

Taken from https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation#Internet_and_IP_basics:

A _CIDR_ of **/24** is the default network size. This corresponds to a subnet mask of _255.255.255.0_, where the last 8 bits are reserved for IP addresses for nodes on a network.

The notation: 192.168.0.2/24 can be interpreted as:

- The _address_ 192.168.0.2
- On the _network_ 192.168.0.0
- With a size of **254** (_2 ^ (32 - 24) - 2_)
    - Usable IPs are in the range 192.168.0.1 - 192.168.0.254
- With a _broadcast address_ of 192.168.0.255
    - In most cases, the last address on a network is used as the _broadcast address_, but this can be changed.

Using this configuration, a device should be able to communicate with any host on the same _network_ (192.168.0.0).

Subnet mask / CIDR of `/24` and `/32` is most common, `/24` being 1 - 254 and `/32` being a single address of 12 digits (`192.168.255.255`)