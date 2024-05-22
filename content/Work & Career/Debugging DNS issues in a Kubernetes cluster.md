Full thread: [https://mattermost.imc.com/imc-fm/pl/ueuw37xqd3gz5kmiedpf34ymxh](https://mattermost.imc.com/imc-fm/pl/ueuw37xqd3gz5kmiedpf34ymxh)

  
On Mon 2023.12.11 it was noticed that several CSVC jobs were failing with some variation of a DNS error and had general, intermittent connectivity issues:

```
Failed to establish a new connection: [Errno -3] Temporary failure in name resolution')
```

Long story short, this resulted from a network switch misconfiguration and was resolved.

### In-depth troubleshooting

I observed that these nodes had issues on them, and had recently joined the cluster as they were rebuilt to Rocky 9.

`chcos0202 chcos0203 chcos0204 chcos0205 chcos0206 chcos0207 chcos0209 chcos0210 chcos0211 chcos0213 chcos0215 chcos0216 chcos0218 chcos0219`

I primarily worked with `chcos0204` as that is where we initially saw issues. I spun up a [debug pod](https://gitlab/mnguyen1/scripts/-/tree/main/k8s-debug-pod "https://gitlab/mnguyen1/scripts/-/tree/main/k8s-debug-pod") on the host & investigated.

Followed these two guides to make sure everything was ok: [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/#create-a-simple-pod-to-use-as-a-test-environment)

https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/#does-the-service-work-by-dns-name

A lot of googling online seemed to suggest that the `coredns` pods for the cluster are overwhelmed & not able to respond to the requests.

This didn’t seem to be the case, as investigating them in namespace `kube-system` did not indicate high resource usage nor any errors in the logs.

Fluent-bit running on the same host seemed to complain about DNS servers, so it wasn’t just our applications that were the issue.

`fluent-bit [2023/12/12 12:18:34] [ warn] [net] getaddrinfo(host='vpc-es-prod-kube-yxamz4crf3c6nzy4bez7bsqt3u.us-east-1.es.amazonaws.com', err=12): Timeout while contacting DNS servers`

Since the DNS lookups were failing intermittently, it was important to isolate the issue down to a particular tuple (source ip:port → destination ip:port) to make it a consistent issue. [IMC Kubernetes Service Troubleshooting | Investigating complaints](https://imcfm.atlassian.net/wiki/spaces/~bewing@trading.imc.intra/pages/324732969/IMC+Kubernetes+Service+Troubleshooting#Investigating-complaints)

The reason to isolate the tuple was that on the source, the application will randomly choose a source port. Based on the tuple, it will take a different path than others.

`bash-5.1# for i in $(seq 1 10); do nslookup kubernetes.default; done ;; connection timed out; no servers could be reached`

Taking a `tcpdump` on the host, outside of the pod to see the traffic. This was run at the same time as the previous command.

```shell
[mnguyen1@chcos0204 ~]$ dzdo tcpdump | grep 10.248.87.109 (pod ip) dropped privs to tcpdump tcpdump: verbose output suppressed, use -v[v]... for full protocol decode listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes 15:31:15.096225 IP 10.248.87.109.48978 > 10.248.83.107.domain: 28634+ A? kubernetes.local.imc-platform-central-services.svc.chco02.ks. (78) 15:31:20.094472 IP 10.248.87.109.48978 > 10.248.83.107.domain: 28634+ A? kubernetes.local.imc-platform-central-services.svc.chco02.ks. (78) 15:31:25.094375 IP 10.248.87.109.48978 > 10.248.83.107.domain: 28634+ A? kubernetes.local.imc-platform-central-services.svc.chco02.ks. (78)`
```

  
We were then able to isolate the tuple:

`10.248.87.109:48978` → `10.248.83.107:53` (port 53 used by default for DNS).

Now we can do a `traceroute` for this tuple:

```shell
bash-5.1# mtr --report -c 100 -L 48978 -u 10.248.83.107 Start: 2023-12-12T22:12:27+0000 HOST: debug Loss% Snt Last Avg Best Wrst StDev 1.|-- 10-204-2-4.p-us-sys-kube- 0.0% 100 0.1 0.0 0.0 0.2 0.0 2.|-- 10.204.2.254 0.0% 100 0.2 0.2 0.2 0.4 0.0 3.|-- 10.253.15.217 57.0% 100 0.2 0.2 0.1 0.3 0.0 4.|-- schco07-e49-1.trading.imc 44.0% 100 0.3 0.2 0.2 0.3 0.0 5.|-- 10-204-7-19.p-us-sys-kube 0.0% 100 0.1 0.2 0.1 1.8 0.2 6.|-- 10-248-83-107.p-us-sys-ku 0.0% 100 0.2 0.2 0.1 0.6 0.1
```



Now we know there’s an issue at the switch `schco07`! Pass over to Network, and we are good now.