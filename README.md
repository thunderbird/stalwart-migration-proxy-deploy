# stalwart-migration-proxy-deploy

This repo contains Kustomize patterns for deploying a Stalwart Migration Proxy on Kubernetes.

## Deployment Documentation

This is documentation for the implementation of the Stalwart Migration Proxy (SMP) at Thundermail. For the ordinary product documentation, see [the Stalwart Docs](https://stalw.art/docs/migration/proxy/). There is also a full [Migration Guide document](https://stalw.art/docs/migration/guide/).

> [!NOTE]
> "Stalwart Migration Proxy" is a lot to type and will be often abbreviated here and elsewhere as "SMP".


### Separation from Stalwart Deployments

One SMP deployment can support routing between any number of backends, enabling not just system migrations, but testing environments and other creative implementations. Because of this one-to-many relationship, it makes sense to deploy SMP separately from Stalwart deployments. To deploy Stalwart itself, rely upon [thundermail-deploy](https://github.com/thunderbird/thundermail-deploy/). There are [new deployment docs](https://github.com/thunderbird/thundermail-deploy/) there to help with that.

> [!NOTE]
>
> SMP only supports the use of a single TLS cert per deployment, meaning that all Stalwart destinations configured in that SMP deployment must be configured to respond to the same domain names. You cannot define multiple certificates and associate them directly with different backends.


## Further Reading

Additional documentation has been written on the following topics:

- [Adding a New SMP Deployment](./docs/new-deployments.md)
- [Adding a New Destination to an Existing SMP Deployment](./docs/new-destinations.md)
