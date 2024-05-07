https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/

```
And the simplest way is to just expose the Docker socket to your CI container, by bind-mounting it with the `-v` flag.

Simply put, when you start your CI container (Jenkins or other), instead of hacking something together with Docker-in-Docker, start it with:

docker run -v /var/run/docker.sock:/var/run/docker.sock 
```