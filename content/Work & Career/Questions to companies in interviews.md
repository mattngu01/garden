- good code health & practices
```
Ask them: "When did you last deploy to production?" and ask them what was involved in doing it, and and how it went. And how often it happens.

A good answer is "This morning. I clicked merge on a small PR, and then watched the tests and and deploys all flow through automatically. The tests make us confident. The automation works. It was uneventful. I'll deploy again whenever I merge another PR."

A Bad answer is "This month. We got signoff from the test team on their manual tests on the latest batch of changes, and management approved the relevant JIRAs, and scheduled a release meeting where we could all carefully monitor the release engineer performing the crucial steps, and then also check the error rate on the live system. The next deploy is scheduled for next month, I hope it goes well, we have a lot of new code in that release!"
```
- how often is your oncall rotation? What is the probability that you get pinged?