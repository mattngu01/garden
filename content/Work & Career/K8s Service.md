Serves similar to an iptable that exposes pod(s). Each pod has an ephemeral ip address, but services expose these pods without having to know these addresses. Instead, it uses a human-readable name.

You can target specific groups of pods using labels, and as such refer to them through the service's name.

This is somewhat similar to [docker compose networking](https://docs.docker.com/compose/networking/) where if you have a container called `web` and `db`, you can refer to the `db` container using the `http://db` url without having to know the container's ip address.

Typically uniquely identifies an application and _can_ 

## Resources
- https://kubernetes.io/docs/concepts/services-networking/service/