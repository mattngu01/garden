**Describe a time when you had to troubleshoot a particularly difficult infrastructure issue. How did you diagnose and solve it?**

At IMC we currently have our Teamcity server managing & coordinating developer builds. We were noticing that builds were not getting scheduled in a timely manner, and any interactions with the server was slowing to a crawl. 

This server is running on a baremetal host. I checked for the typical issues that can come up: CPU usage, RAM availability, disk space, and network connectivity. 

The CPU, RAM usage, and network traffic had decreased significantly from baseline. Disk space was close to full on a particular partition where the DB was hosted.

I increased the partition by a few GB to relieve disk pressure and give some time. Since the contents of the DB were only a 1-2 hundred GB and the allowance for downtime was a few minutes, I announced a maintenance window since users couldn't do anything anyways and moved the DB to a new parition, and symlinked the folder to the new partition.

Later that weekend I worked with the rest of my team to move this DB to a separate server.


**Explain the principle of Infrastructure as Code (IaC). How have you implemented it in your past projects?**

Infrastructure as Code is the concept of storing server configuration and various data center objects through code. Doing so manually is prone to mistakes and long turnaround times. IaC shortens the deploy time, makes it less error prone, and frees up developer time.

**How do you handle database migrations in a CI/CD pipeline without causing downtime?**

Staging the database migrations would be ideal in avoiding downtime. For example, removing a column would require:
- dropping the column
- removing all code references to the column

Making the desired column nullable would be paired with removing all code references to the column.

Separately, after all references is gone, the column would be dropped.

Doing the first stage is a low effort & impact way to evaluate the work needed to be done to remove the code references, while still supporting new features. If there is any issues, the first stage can be rolled back easily.

**Describe the differences between blue-green deployments and canary releases. Which do you prefer and why?**

Blue-green deployments and canary releases both gradually test new changes. Blue-green deployments typically are done by gradually directing network traffic to a particular deployment that is different. Canary releases are apps / software deployed to a small subset of hosts / machines to test for any issues.

Canary releases are in relation to software releases, whereas blue-green deployments are about deployment environments. Blue-green deployments can also have new software releases though.

While both are adequate tools for testing new changes, I would prefer blue-green deployments as it is a broader set of things you can change, whether it be network configuration or a new app version.

**How would you handle monitoring and logging for a microservices architecture?**

Monitoring can be done with service discovery of each service, and standardizing metrics that are being displayed for each type of service. This can then be aggregated into a collector & stored in a time series database for monitoring & alerting.

For logging, a similar structure can take place. Each service can have a log forwarder tail container that sends logs from the service to an aggregator, storing it in an appropriate place.

**Explain a situation where you had to collaborate with a development team to improve a CI/CD process.**

I work with developers frequently to help improve build speeds. The culprit can be a variety of things.

I worked with a developer on their particular build where it ran an upgrade pipeline, and it was critical that it completed successfully within a 30 minute time frame. I set out to understand what exactly their build did, which boiled down to build & compile, and transfer the artifact directly to several deployments. 

This was resolved by working with them to slim down the Docker container that ran the build, as well as uploading their artifact to an artifact store that had better bandwidth and resources to fan out artifacts.

**What are your considerations when designing a scalable and secure cloud infrastructure?**

Some considerations are:
- cost & efficiency of scaling
- access control for all aspects of infrastructure lifecycle (spinning it up, accessing, tearing it down)

**Which tools or technologies have you used for container orchestration, and why did you choose them?**

I have used Kubernetes paired with Helm for container orchestration. I chose this as these are common tools used across stacks.

**How do you handle secrets and sensitive data in your CI/CD pipelines and deployments?**

This is handled by a vault similar to Hashicorp Vault, where secrets and sensitive data can be queried instead of stored insecurely. The CI/CD pipeline can query over a secure connection and it will only use the token as needed. This also allows for strict access control.

**Describe your experience with automating infrastructure provisioning and configuration management. Which tools have you used?**

I have not had much experience with IaC tools. I am curious and am able to learn how to use whatever tools available to automate infrastructure provisioning.

**How do you ensure that the infrastructure you set up is resilient to failures?**

One of the easiest ways to increase failure resilience is with redundancy. Having multiple servers, application frontends, etc where traffic can be switched from one to another in case of failure is an easy way to make it resilient.


**In your opinion, what's the biggest challenge in the DevOps field today? How would you approach it?**

The biggest challenge is maintaining connection and communication with developers. This can be done in a variety of ways: surveys, ad-hoc talks, case studies.

I would approach by doing a quarterly, official survey for developers on pain points to prioritize large issues. In addition, keeping track of various incidents that cause developers to reach out to operation teams can help with getting patterns of problems.