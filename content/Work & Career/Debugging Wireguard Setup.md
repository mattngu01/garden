## Context
At this point you have two apps that are configured incorrectly. The client is talking directly to the server over the [Fly.io private network (6pn)](https://fly.io/docs/reference/private-networking/). This is what you'd normally want to do on Fly.io, but this is a hiring challenge so we're going to do it the weird way. What we want is for the client to talk to the server using a WireGuard tunnel, which is set up (badly) on both instances.

There's a `test-connection.sh` script that tries to connect to the server from the client first over 6pn, and then over wireguard. The end result should be that connections over 6pn should be blocked, and connections over wireguard should work. The opposite is the case right now. The test script is only present on the client machine since you don't need to make it work in the other direction.

The 6pn addresses for this are the `[fdaa::]` addresses on `eth0` in each vm. WireGuard should be running over 6pn, using the `fdaa` addresses. That's fine! It's what we want. We just don't want to use the 6pn addresses directly for HTTP; we want to route them over a WireGuard connection running over our 6pn addresses.

## Summary of changes

Issues pertaining to this setup can be boiled down into two parts, respectively for server and client:
- Wireguard configuration
- Host firewall configuration

High-level changes made:
- wireguard configuration
   - MTU mismatched settings, client -> too large MTU, server -> should not be lower than its clients
   - add 6pn endpoints
   - change peer allowed ip's, they were configured for its own subnets, not the host on the other end
- firewall changes
   - remove ipv6 firewall rules that were dropping UDP packets on port 51820
   - remove ipv4 firewall rule dropping all TCP
   - keep ipv6 TCP traffic on port 22, but REJECT the rest
- kernel config
   - allow ICMP echo

## Verifying Wireguard configurations

### Client

- Remove MTU setting to wireguard, ethernet interface only supports up to 1420
   - wireguard set it to a MTU of 1340
- add the server 6pn to the endpoint
- maybe add persistent keepalive? Our later firewall configurations don't need to enforce persistent connections
- add `192.168.0.1` peer allowed IP

```shell
root@2865101ae21328:/etc/wireguard# diff wg0.conf wg0.conf.backup --side-by-side
[Interface]                                                     [Interface]
PrivateKey = +IZaOYtIJVaWnDp88i9r53utvlQymZDIcEBqCmll6FE=       PrivateKey = +IZaOYtIJVaWnDp88i9r53utvlQymZDIcEBqCmll6FE=
Address = 10.0.0.2/24                                           Address = 10.0.0.2/24
ListenPort = 51820                                              ListenPort = 51820
                                                              > MTU = 1800

[Peer]                                                          [Peer]
PublicKey = g9stvuZgnSYtvx+jbtpwfNEpmDf2NzrlZRHRvmBnk1s=        PublicKey = g9stvuZgnSYtvx+jbtpwfNEpmDf2NzrlZRHRvmBnk1s=
Endpoint = fdaa:9:1e22:a7b:a15f:498e:f263:2:51820             / AllowedIPs = 10.0.0.1/32
AllowedIPs = 192.168.0.1/32                                   <
```

### Server

- add client 6pn endpoint
- removed MTU setting for good measure, we can let Wireguard figure it out
- change peer allowed ip to client interface ip (`10.0.0.2/32`)


```shell
root@d891260b6d0418:/etc/wireguard# diff wg0.conf wg0.conf.backup --side-by-side
[Interface]                                                     [Interface]
PrivateKey = 4A08znaNK3mRR3LQJPx6/BumVuLYrNrseK5ORUnbXn8=       PrivateKey = 4A08znaNK3mRR3LQJPx6/BumVuLYrNrseK5ORUnbXn8=
Address = 192.168.0.1/24                                        Address = 192.168.0.1/24
ListenPort = 51820                                              ListenPort = 51820
                                                              > MTU = 1280

[Peer]                                                          [Peer]
PublicKey = Khoh2j2WX389oqFzAvumHv+X7+dtPr1aSEttQHvkxC8=        PublicKey = Khoh2j2WX389oqFzAvumHv+X7+dtPr1aSEttQHvkxC8=
Endpoint = fdaa:9:1e22:a7b:16f:a63a:aa21:2:51820              | AllowedIPs = 192.168.0.2/32
AllowedIPs = 10.0.0.2/32                                      <
```

Additional things checked, but didn't require 
- double checking public keys matched in `[Peer]` sections, make sure private/public key pairs are correct

## Firewall configuration

At this point, we can verify that the wireguard configuration is correct, and make sure our desired traffic is actually getting out and/or being received.

Attempting to get the route to `192.168.0.1` looks good:

```shell
root@2865101ae21328:/etc/wireguard# ip route get 192.168.0.1
192.168.0.1 dev wg0 src 10.0.0.2 uid 0
    cache
```

Sidenote, there's a separate route for the 192.168.0.0/24 subnet but since there's a preceding route, any requests to `192.168.0.1` should go to interface `wg0` still:
```shell
root@2865101ae21328:/etc/wireguard# ip route
default via 172.19.6.33 dev eth0 proto static
10.0.0.0/24 via 172.19.6.34 dev eth0
10.0.0.0/24 dev wg0 proto kernel scope link src 10.0.0.2
172.19.6.32/29 dev eth0 proto kernel scope link src 172.19.6.34
192.168.0.0/24 via 172.19.6.34 dev eth0
192.168.0.1 dev wg0 scope link
```

Attempting a `ping 192.168.0.1` & watching traffic on interface `wg0` seems to send the ICMP request, with no response. 

```shell
root@2865101ae21328:/# tcpdump -i wg0
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wg0, link-type RAW (Raw IP), capture size 262144 bytes
22:59:24.767429 IP 10.0.0.2 > 192.168.0.1: ICMP echo request, id 540, seq 28, length 64
```

Also using `tcpdump`, there aren't any UDP packets going out to our server. Let's check the firewall to make sure we're not doing anything:

```shell
# since we're using 6pn addressing, let's check the ipv6 firewall. TIL ip6tables existed!
root@2865101ae21328:/etc/wireguard# ip6tables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    DROP       udp      anywhere             anywhere             udp dpt:51820

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
# we're dropping UDP traffic!
root@2865101ae21328:/etc/wireguard# ip6tables -D INPUT 1
```

Despite dropping it, still no response. Let's check the firewall on the server:

```shell
root@6e82950c160d87:/etc/wireguard# ip6tables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    DROP       udp      anywhere             anywhere             udp dpt:51820
# truncated, same issue here though!
```

Attempting to send traffic again, `wg` shows that the handshake works! We can certainly check and see if the server is receiving the UDP packets with `root@6e82950c160d87:/usr/share/flylo-world# tcpdump -v 'udp and src fdaa:9:1e22:a7b:16f:a63a:aa21:2 and port 51820'`. 

Note: for `tcpdump`, we will see the received packets _before_ they hit the firewall. This is the opposite for outbound packets. Keep this in mind when using `tcpdump` to solely verify if we're correctly receiving packets or not.

```shell
# on client, server will also show successful handshake
root@2865101ae21328:/etc/wireguard# wg
... truncated...

peer: g9stvuZgnSYtvx+jbtpwfNEpmDf2NzrlZRHRvmBnk1s=
  endpoint: [fdaa:9:1e22:a7b:a15f:6269:cd8e:2]:51820
  allowed ips: 192.168.0.1/32
  latest handshake: 42 seconds ago
  transfer: 220 B received, 25.46 KiB sent
```

Ping still doesn't seem to have any returns, and `curl` doesn't seem to be working. Checking the server IPv4 firewall:

```shell
root@6e82950c160d87:/etc/wireguard# iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    DROP       tcp  --  anywhere             anywhere             tcp dpt:http-alt

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
root@6e82950c160d87:/etc/wireguard# iptables -D INPUT 1
```

```shell
## good to go for curl!
root@2865101ae21328:/etc/wireguard# curl 192.168.0.1:8080
Hello!

This is the Fly SRE network work sample test file. This should be accessible over the wireguard network configured between the two work sample apps and not directly over the 6pm private network (fdaa::).
```

## Server Kernel ICMP block

[Googling on what can cause a ICMP ping to not be acknowledged](https://unix.stackexchange.com/questions/412446/how-to-disable-ping-response-icmp-echo-in-linux-all-the-time) on a host comes up with two options: either w/ `sysctl` or with firewall rules. We've ruled out the firewall after looking at it earlier as it should be accepting/sending all traffic on both ends, so let's try setting the flag listed in the SO post.

```shell
root@6e82950c160d87:/etc/wireguard# vim /etc/sysctl.conf
...rest of file...
# do not ignore ICMP echo
net.ipv4.icmp_echo_ignore_all=0

root@6e82950c160d87:/etc/wireguard# sysctl -p
net.ipv4.icmp_echo_ignore_all = 0
```


## Blocking HTTP traffic on 6pn traffic not going through wireguard

Last thing to do is to block HTTP traffic over 6pn directly. Let's add two firewall rules to the server, since we don't want to use HTTP on 6pn:

```shell
# I still want to be able to SSH in
root@e784935c014983:/usr/share/flylo-world# ip6tables -A INPUT -p tcp --dport 22 -j ACCEPT

# but let's block the rest of the traffic
root@e784935c014983:/usr/share/flylo-world# ip6tables -A INPUT -p tcp -j REJECT
```

Running `test-connection.sh` passes now.


## Miscellaneous Notes / Links / Raw Notes
- expecting user root for ssh
- https://www.wireguard.com/#conceptual-overview
- I tried installing `ufw`, but that seemed to bork things so I used this opportunity to learn about `iptables`


local wireguard interface uses ip6

Things I did:
- add 6pn endpoints for respective peers
- need to match ip range of server
- keepalive
- MTU for wireguard interface too high? the actual outbound interface is 1420
- drop iptables for tcp connections on server
- `net.ipv4.icmp_echo_ignore_all = 0` in `/etc/sysctl.conf`, enable icmp pings
- `ip6tables -A INPUT ! -i wg0 -j REJECT`, reject things that are outside of the wireguard connection
- there was existing ip route for 192.168.0.1/24 on client, removed it and flushed
