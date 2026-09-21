# stalwart-migration-proxy-deploy

This repo contains Kustomize patterns for deploying a Stalwart Migration Proxy on Kubernetes.

## Deployment Documentation

This is documentation for the implementation of the Stalwart Migration Proxy (SMP) at Thundermail. For the ordinary product documentation, see [the Stalwart Docs](https://stalw.art/docs/migration/proxy/). There is also a full [Migration Guide document](https://stalw.art/docs/migration/guide/).

> [!NOTE]
> "Stalwart Migration Proxy" is a lot to type and will be often abbreviated here and elsewhere as "SMP".


### Separation from Stalwart Deployments

One SMP deployment can support routing between any number of backends, enabling not just system migrations, but testing environments and other creative implementations. Because of this one-to-many relationship, it makes sense to deploy SMP separately from Stalwart deployments. To deploy Stalwart itself, rely upon [thundermail-deploy](https://github.com/thunderbird/thundermail-deploy/). There are [new deployment docs](https://github.com/thunderbird/thundermail-deploy/) there to help with that.


### Adding a New Destination

In migration proxy lingo, a "destination" is a Stalwart deployment which is configured to receive mail traffic with [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) headers from this SMP deployment. There are many valid ways to configure Stalwart beyond that, but to add a new destination, you'll need to do at least these things:

1. Set up your Stalwart installation to permit network access to its mail service from your SMP deployment. Typically, this should be private network access, but that is not technically required. If you use thundermail-deploy, you can set `use_migration_proxy` to `true` when you [prepare supporting AWS resources](https://github.com/thunderbird/thundermail-deploy/blob/main/docs/new-deployments.md#prepare-supporting-aws-resources), and set `migration_proxy_sgid` to the security group that gets attached to your SMP pod.

2. In your Stalwart destination's configuration, trust the SMP's network as a proxy network. This is part of the `SystemSettings` object in your Stalwart deployment, which you can find in the web console in the Settings → Network → General. Use the CIDRs that the SMP is deployed on. For example, to trust proxying from SMPs running on our tb-dev "general" nodes, you can run:

```bash
stalwart-cli update SystemSettings singleton \
    --field 'proxyTrustedNetworks={"10.120.64.0/18": true, "10.120.128.0/18": true}'
```

3. Set up a TLS secret in Kubernetes containing the cert and key for any domains your Stalwart deployment is going to handle. Create a reference to this secret in your deployment overlay, a la [the tb-dev deployment](https://github.com/thunderbird/stalwart-migration-proxy-deploy/blob/main/overlays/tb-dev/deployment.yaml).

4. Add to the SMP configuration file in your deployment's ConfigMap overlay a destination representing your new Stalwart deployment. This needs to be configured to use the SSL cert and key you added to the deployment in the last step. Deploy these changes to the live SMP installation.

5. Point your mail domain to the SMP's access point.

6. If you have made your deployment the default destination in the SMP config, you can test with a brand new account in your brand new deployment. If not, you will still have to create the new account, but you will also have to instruct SMP to map that account to your new destination. Documentation on using the SMP API can be [found here](https://github.com/thunderbird/thundermail-deploy/blob/main/docs/migration-tools.md#smp-api).

You should now be able to test connectivity to your mail server using that account.
