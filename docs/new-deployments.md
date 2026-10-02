# Adding a New SMP Deployment

You may desire to serve email from behind a Stalwart Migration Proxy from a domain not currently covered by other SMP deployments. Because SMP cannot serve multiple certificates, associating them directly with different backends, in order to serve multiple domains behind a single SMP deployment, you must use a certificate that covers all of the domains. This is *not* the intended usage model, though. Ideally, you want one SMP deployment per domain you serve mail from.

> [!NOTE]
>
> This is not to be confused with the notion of using custom domains for mailboxes on a single primary domain.


## Prepare supporting AWS resources

> [!NOTE]
> This section assumes some knowledge of the platform-infrastructure repo. Please familiarize yourself with at least its Pulumi components prior to running these steps.

Before you can run out any manifests to Kubernetes, you'll need to build a bunch of resources in AWS (like security groups, a Redis instance, etc.) first to support that deployment. This is done through a Pulumi module called [stalwart_migration_proxy_deploy](https://github.com/thunderbird/platform-infrastructure/blob/main/pulumi/modules/stalwart_migration_proxy_deploy.py). You'll need to set up an instance of this module prior to making any Kubernetes changes.

You'll need a call in your environment's `__main__.py` file to `stalwart_migration_proxy_deploy.create_application_dependencies`. If your target cluster already has a stalwart-migration-proxy-deploy installation in it, this is probably already done. Double-check the code to make sure it's written in such a way that you can add a config section and have that picked up by the provisioning code. This configuration should be set off by a unique deployment name, which you'll want to reference elsewhere; make that decision now.

Then you'll need a configuration in your environment's `config.prod.yaml` file. The options are fully documented in the Pulumi module, but here is a stub configuration you can fill out:

```yaml
  deployment_name: # Unique deployment name that will be a part of every resource's name
    kubelet_probe_source_sgid: # SG of your cluster's control plane components
    private_subnet_ids:
      - # Private subnet ID for first AZ
      - # Private subnet ID for second AZ
    redis_name: # Name of your Redis instance
    services: ['all'] # List of mail/API services to enable in Stalwart, or "all" for a full set
    redis_nodes: 1 # Increase for production envs
    redis_node_type: cache.t3.micro # Make this bigger for production envs
```

You can verify the resources slated to be built with a `pulumi preview`. When it's ready, open a PR, get it approved, and merge it. Post-merge actions on platform-infrastructure will run the `pulumi up` for you, creating the resources you need.

You will eventually need the details of various resources built by this operation, such as the IAM role ARN, the connection address of the Redis instance, security group IDs, etc. These will go into the Kustomize overlays for your deployment a little later down the line.


## Prepare SSL secrets

You'll need an SSL certificate that covers the domain that this SMP and the Stalwart destinations behind it operate on. If you don't have one, follow our guides. You'll need to complete all of these steps:

- [Create a cert](https://github.com/thunderbird/thundermail-deploy/blob/main/docs/configuration.md#create-and-validate-an-ssl-certificate)
- [Export the cert](https://github.com/thunderbird/thundermail-deploy/blob/main/docs/configuration.md#export-and-format-the-ssl-certificate-for-stalwart-and-nginx)
- Create a Kubernetes secret for this cert in SMP following [this guide for nginx](https://github.com/thunderbird/thundermail-deploy/blob/main/docs/configuration.md#using-the-certificate-in-nginx), except in your SMP namespace


## Create AWS Secrets Manager secrets

A pattern used here (and elsewhere) is to begin with handmade secret resources using AWS Secrets Manager, then the External Secrets Operator (ESO) will come along later and create Kubernetes Secret resources out of them. For this repo, you need just one:


### SMP API Bearer Token

The migration proxy's API is authenticated using a Bearer token which you must generate using standard secure practices. Create a Secret in AWS to store it:

```bash
aws --profile $AWS_PROFILE secretsmanager create-secret \
  --name "mzla/$ENVIRONMENT/$NAMESPACE/stalwart-migration-proxy-bearer-token" \
  --secret-string "{\"token\": \"$SECRET\"}"
```


## Copy the Kustomize template

There's a Kustomize template ready for you to clone and fill out, so copy it into a new folder named after your deployment name from earlier.

```bash
cp -r overlays/{_template,$DEPLOYMENT_NAME}
```

## Update the Kustomize template

The template has several fields you will have to fill out. The previous Pulumi steps will have built out a series of resources whose IDs you will need to populate throughout your overlays. To ensure you fill everything out, the template you copied has the term `SETUP:` next to every field to populate or decision to make. All of these decisions are documented alongside the `SETUP:` term with instructions on how to adapt the overlay to your needs.

Run through all of these, (`grep -rn 'SETUP:' overlays/$DEPLOYMENT_NAME/` if you like) setting the right values where necessary, deleting the `SETUP:` term as you make each change.

Double-check that you have produced valid manifests by building your project:

```bash
kustomize build overlays/$DEPLOYMENT_NAME
```

Push your changes up. If you are not on a working branch, you'll need to go through with a merge to `main`. Allow ArgoCD to build the resources you've requested. Any problems you encounter at this point are beyond the scope of this documentation; you'll have to debug them as they arise.


## Create an ArgoCD application

Pull the latest commits in the [platform-infrastructure repo](https://github.com/thunderbird/platform-infrastructure/) and branch it. Find an existing Argo Application manifest for this repo (such as the one at `argocd/tb-dev/apps/stalwart-migration-proxy.yaml`) and copy it into the appropriate cluster application directory for your deployment.

- Make any adjustments to the relevant Project manifest to make sure your app is deployable to your chosen cluster and namespace.
- Update this Application manifest:
    - Make sure it has a unique app name.
    - Ensure the `project` reference is correct.
    - Point the Kustomize overlay `path` to the overlay you just created.
    - Ensure the destination server is accurate.
    - Update the destination `namespace` field to the one you decided on before.
    - If you like, you can set the source's `targetRevision` to a working branch while you do the initial buildout.

Create a PR with these changes. Get it reviewed and merged. You can either wait out the auto-sync period (<1 hour), or you can do the relevant refreshes (`argocd-projects` if you made changes to the project file, and the app-of-apps project for your deployment target). Since your overlay hasn't been updated and pushed, this should create your application, but that application should have deployment errors. We expect that right now.


