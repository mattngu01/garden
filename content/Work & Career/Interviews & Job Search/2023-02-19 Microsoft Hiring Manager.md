Matt Bearup - Linux Packaging

[mbearup@microsoft.com](mailto:mbearup@microsoft.com "mailto:mbearup@microsoft.com")

Hobbies
- minecraft add-ons -> magic
	- mod
- making own video games
	- dark souls, but lighter in tone
- sunny in philadelphia - main chars are not great people

Working @ Microsoft
- push to be modern & competitive
- don't push the people too hard
- benefits are good
	- 1500 on health & wealth
- Microsoft not gonna push in-person

Azure Core
- Linux packaging (ingest API, they send RPMs & RPM distribution)
	- all internal
	- mariner / Azure Linux
	- 28 regional mirrors
	- Azure Frontdoor - global cache for content delivery
		- replicate on mirrors all around the globe
		- caching strategy
		- deletes very quickly
	- backend Django Storage - in multiple storage backends
	- Capture checksum blob storage
	- Corellation w/ files in DB
	- next project
		- enable authentication
		- mirror 3rd party content
		- package building pipeline (tool like `fpm`)
		- expanding package formats
	- code vs infra 
	- oncall hours
		- multiple rotations, paired with provisioning
		- 1 week, then 2-3 month break
		- half a week, then
		- summer have full staff
		- 24h oncall
		- follow the sun? not really
	- incidents
		- business hours sev 3 & sev 4
		- sev 2 - 24/7
			- call to dial in
			- call another person if can't do it
		- more sporadic, can go weeks without it
	- 5-6pm PST security team will drop sev 2 on something they want done
	- US time zones are fine, majority of engineers were in EST
	- focus areas
		- CDN
		- content API
		- infrastructure
	- storage platform Azure
- provisioning (Linux VM in Azure)
	- avg sev 2 per week
	- projects
		- platform changes
		- VM - how to get init config
			- get it from iso from http endpoint
	- book a meeting with all of them?
- Community Engineering - (open source contribution)
	- pulp
	- systemd
	- no oncall

pulp
a lot of python
FastAPI

Kubernetes
- prebaked deployment system
- configmap patch

felt good about it, he said it was clear that I was a "expert" (or something like that) on the subject matter based on the conversation we had and what we were talking about.

did small coding problem, anagram, but he said i did fine /shrug, did not do the best tbh since i didn't expect it but yeah

good questions to ask:
- what was the last outage and how was it resolved?
- what does your CI/CD look like?