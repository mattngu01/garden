https://news.ycombinator.com/item?id=15326055

> Another benefit is that it allows better fine-grained control over _when_ metrics gathering is done, and _what_. Since Prometheus best practices dictate that metrics should be computed at pull time, it means you can fine-tune collection intervals to specific metrics, and this can be done centrally. And since you only pull from what you have, it means there can't be a rogue agent somewhere that's spewing out data (i.e. what a sibling comment calls "authorative sources").

> But to understand why pull is a better model, you have to understand Google's/Prometheus's "observer/reactor" mindset towards large-scale computing; it's just easier to scale up with this model. Consider an application that implements some kind of REST API. You want metrics for things like the total number of requests served, which you'll sample now and then. You add an endpoint /metrics running on port 9100. Then you tell Prometheus to scrape (pull from) [http://example.com:9100/metrics](http://example.com:9100/metrics). So far so good.

> The beauty of the model arises when you involve a dynamic orchestration like Kubernetes. Now we're running the app on Kubernetes, which means the app can run on many nodes, across many clusters, at the same time; it will have a lot of different IPs (one IP per instance) that are completely dynamic. Instead of adding a rule to scrape a specific URL, you tell Prometheus to ask Kubernetes for all services and then use _that_ information to figure out the endpoint. This dynamic discovery means that as you take apps up and down, Prometheus will automatically update its list of endpoints and scrape them. Equally importing, Prometheus goes to the _source_ of the data at any given time. The services are already scaled up; there's no corresponding metrics collection to scale up, other than in the internal machinery of Prometheus' scraping system.


```
Because it shifts all the complexity to the monitoring system, making the "agents" really, really dumb. There would have to be more to push than just a single IP:

* Many installations run multiple Prometheus servers for redundancy, so to start, it'd have to be multiple IPs.

* They would also need auth credentials.

* They'd need retry/failure logic with backoff to prevent dogpiling.

* Clients would have to be careful to resolve the name, not cache the DNS lookup, in order to always resolve Prometheus to the right IP.

* If Prometheus moves, _every_ pusher has to be updated.

* Since Prometheus wouldn't know about pushers, it wouldn't know if a push has failed. As Prometheus is pull-based, you can detect actual failure, not just absence of data.
```