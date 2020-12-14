# flux2
Project with a Helm Chart deploying a simple echo service, meant to be used with Flux2.

### Prerequisites

Kubernetes cluster with [Flux](https://github.com/fluxcd/flux) installed and `fluxctl`.

### Git repository
`GitRepository` defines the source that contains the chart.
Trusted sources will be registered by a cluster administrator by creating the resources in the flux-system namespace.

```
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: echo
spec:
  interval: 30s # Defines at which interval the Git repository contents are fetched
  url: https://github.com/gonzalobarbitta/flux2
  ref:
    branch: main
```

Or, using `fluxctl`

```
flux create source git echo \
  --url=https://github.com/gonzalobarbitta/flux2 \
  --branch=main \
  --interval=30s \
  --export > ./source.yaml
```

### Kustomization
Then, we create a `kustomization` for synchronizing the manifests on the cluster.

```
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: echo
spec:
  interval: 1m
  path: ./releases
  prune: true
  sourceRef:
    kind: GitRepository
    name: echo
```

Or, using `fluxctl`

```
flux create kustomization echo \
  --source=echo \
  --path="./releases" \
  --prune=true \
  --validation=client \
  --interval=1m \
  --export > ./kustomization.yaml
```

### Helm Release

One of these manifests will be the Helm Release, which references the chart source (Git repository).

 ```
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: echo
  namespace: echo # Namespace where the chart will be installed
spec:
  interval: 1m
  targetNamespace: echo # Namespace in which the chart components will be installed
  chart:
    spec:
      chart: ./charts/echo # Path inside the git source where chart is located
      sourceRef:
        kind: GitRepository
        name: echo
      interval: 1m
 ```