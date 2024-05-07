## What do you do at IMC?
- work at Central Services
- there are various applications, infrastructure that we overlook & maintain
- these can vary from the build server & queue distribution to package repositories
## Stories and STAR

### Revamp & Centralize Monitoring System
- We have various monitoring systems & stacks varying across services, hard for newcomer & ppl working adjacent to us to get familiarized 
- recognized that users would often detect issues before we did & we were missing critical parts of monitoring, replicating user actions 
- task to centralize monitoring systems & add in this critical part
- Created distributed task queue to replicate user actions & created dashboard to view this data, along with bringing together our dashboards in a digestible manner
- result: reduce resolution time from ~30 minutes (majority of time was not being aware of issue) to ~5 minutes 
	- original goal was to notice issues before users do 

### Creating an RPM when I first started at IMC
- we keep in-house RPM packages, which are vanilla packages created with additional company config & patches (maven)
- Task was to bump version in underlying package, remove a patch
- ended up breaking a large majority of builds, RPM was extract to wrong directory due to standards changing and the original way to build it very old (used a makefile on a computer)
- after realizing this, fixed it & ended up simplifying workflow by containerizing it
- now anyone in the company can safely build their RPM & reduced load on the team, more consistent way to build RPM's


### PyPi migration PPL conflict
- migration from baremetal server w/ unsupported app to company standard on K8s
	- despite being a critical piece of infrastructure, it was very unstable and had issues constantly
- did migration on weekend, got called in on Saturday nights
- ppl builds were not working bc we missed an implementation detail that a legacy process used, but the process was super outdated 
- colleague was upset, helped defuse the situation saying we can roll it back & go back to the drawing board
- came back to the next week & now we are happily on K8s with much more stability

### Docker Registry synchronization tool
- we migration docker to new global infrastructure & off of legacy application 
- we have sync natively, but does not retry. Build tool to keep registries in sync bc once they do it's not good
- created multithreaded Python job to diff contents of all registries & sync
- successfully run global infrastructure in Amsterdam & Sydney, not saturating bandwidth between continents


## Strengths & Weaknesses
- strengths
	- communication & documentation? -> garner understanding
- weaknesses
	- getting to bottom of what we need to solve for problems: getting the problem statement & focusing in on we need to solve
		- sometimes I end up solving the symptoms, but not the root cause
		- getting better at this, and I think the monitoring system is a very good example of that

## Questions to ask
- what is the on-call rotation like & how often do you actually get paged for it in a given cycle?
- what percentage would you say that you are coding vs. doing operational work?
- What do you like about your company?
- What do you expect out of someone, in the first 6 months of hiring?
	- consistently looking to do a little more (not repeating same pattern)
	- 


in growth, if i were to propose an idea then how would that be received? 



