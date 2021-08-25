**Welcome, stranger!**

You arrived at a Kubernetes cluster repository.

It contains a list of cluster resources managed by [Flux CD](https://fluxcd.io/) GitOps tool.

To add a new resource to the cluster, feel free to submit a PR for `master` branch.

Resources can be defined as:

* Raw YAML manifests:

        apiVersion: extensions/v1beta1
        kind: Deployment
        metadata:
          labels:
            app: nginx
          name: nginx
          namespace: monitoring
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: nginx
          template:
            metadata:
              labels:
                app: nginx
            spec:
              containers:
              - image: nginx
                name: nginx

* `HelmRelease` CRD manifest for self-hosted charts:

        apiVersion: helm.fluxcd.io/v1
        kind: HelmRelease
        metadata:
          name: default-namespace-policies
          namespace: flux   # We put HelmRelease resources to the same namespace as Flux
        spec:
          releaseName: default-namespace-policies
          targetNamespace: monitoring   # The namespace where Helm chart gets deployed
          chart:
            git: https://github.com/artazar/helm     # Git repository URL where Helm chart is stored
            path: security/default-namespace-policies   # The path to the chart on the repository
            ref: master                                 # Use custom branch if needed
            secretRef:
              name: flux-git-auth   # This should be added for the Helm charts on our internal Git repositories to fetch credentials
          values:                   # The list of Helm values goes here
            allow_internet: true

* `HelmRelease` CRD manifest for public charts:

        apiVersion: helm.fluxcd.io/v1
        kind: HelmRelease
        metadata:
          name: nginx
          namespace: flux   # We put HelmRelease resources to the same namespace as Flux
        spec:
          releaseName: nginx
          targetNamespace: flux-test   # The namespace where Helm chart gets deployed
          chart:
            repository: https://charts.bitnami.com/bitnami
            name: nginx
            version: 7.1.2
          values:
            fullnameOverride: nginx
            replicaCount: 2
            image:
              tag: 1.19.3

* [Kustomization](https://kustomize.io/) files

Put your resources inside directories, named after namespaces, avoid creating new files on the repository root.

The resources should appear in the cluster (or disappear if getting removed) within 10 minutes after merging to master.

In case this does not happen, check Flux logs for any errors:

    # kubectl -n flux logs deploy/flux
    # kubectl -n flux logs deploy/helm-operator

### Pre-commit

This repository uses [Pre-commit](https://pre-commit.com/#intro) hooks.

To use them locally, run the following commands:

    # pip install pre-commit
    # pre-commit install
