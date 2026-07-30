# stalwart-migration-proxy-deploy

This repo contains Kustomize patterns for deploying a Stalwart Migration Proxy on Kubernetes. This is deployed separately from the other Stalwart components because there is a one-to-many relationship between them. We may want to have multiple Stalwart installations across clusters and namespaces, but only one migration proxy routing traffic between them.
