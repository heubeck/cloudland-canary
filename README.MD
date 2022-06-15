![Title](_img/title.png)

# Automated Canary Deployment Example

Setup uses:

* **K8s Kind** as local Kubernetes environment
* **FluxCD** and a **GitHub** repository for managing the cluster
* **Linkerd** service mesh for traffic management and observibility
* **Flagger** for reliable releasing
* **Prometheus** because there's no reason for not using it
* **hey** to get some traffic
* **𝕰𝖝𝖆𝖒𝖎𝖓𝖊𝖗** just another simple test app

## Setup

To reproduce the example project please walk through the following steps

### Fork and clone the example

Create a fork of this repository, clone it to your local machine and change into its directory.
The following is expected to be executed within there.

### Local k8s kind cluster

[Install k8s kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) or use the Kubernetes distribution you prefer.

```shell
# Create new cluster and configure kubectl
kind create cluster --config ./_kind/config.yaml --image=kindest/node:v1.24.0
kind export kubeconfig --kubeconfig $HOME/.kube/config
```

### Flux

[Install flux cli](https://fluxcd.io/docs/cmd/#install-using-bash) for bootstrapping FluxCD, the GitOps operator synchronizing the K8s manifests into your cluster.

```shell
# Initialize the cluster and configure synchronization from git repository
export GITHUB_TOKEN= # your github personal access token with repository scope
export REPO_OWNER=heubeck # replace with your github account
export REPO_NAME=cloudland-canary
flux bootstrap github \
  --owner=$REPO_OWNER \
  --repository=$REPO_NAME \
  --private=false \
  --personal=true \
  --path=bootstrap \
  --branch=main
```

The bootstrap command may vary on a different constellation, like with an organization or non-public repostory. Please check the Flux documentation.

It will last some minutes for the cluster components to become ready. You can check for completion and validate the setup by looking into Flux' Kustomization resources:

```shell
kubectl get kustomization -A
```

### Linkerd

The linkerd cli is not mandatory, but eases access to its dashboard and helps with troubleshooting.

* [Install linkerd cli](https://linkerd.io/2.11/getting-started/#step-1-install-the-cli)
* [Install linkerd smi extension](https://linkerd.io/2.11/tasks/linkerd-smi/)

### hey

[hey](https://github.com/rakyll/hey) is a simple but efficient load producer.

### Access components

* [Examiner](https://github.com/heubeck/examiner) - Bound to ingress: GET http://localhost/examine
* Prometheus - Port forwarding: `kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090`
* Linkerd Viz - Port forwarding: `kubectl port-forward -n linkerd-viz svc/web 8084:8084` or via cli: `linkerd viz dashboard`

## Overview

### Ordinary setup without canary deployment automation

![Basic Setup](_img/Flaggerless.png)

### Enhanced setup for automated canary deployments

![Canary Setup](_img/Flaggerfull.png)

## Walkthrough

The following steps will demonstrate the example setup.

### Look what we got

```shell
kubectl  get deployment -A
```

### View the linkerd dashboard

```shell
# Creates some port-forwards and blocks, run in a separate shell
linkerd viz dashboard
```

### Let some requests flow

```shell
hey -c 5 -q 5 -z 5m "http://localhost/examine?delay=100..250"
```

And explore the linkerd dashboard.

## Cleanup

```shell
# Stop and remove the kind cluster
kind delete cluster
```

---

Congratulations. You made it to the end. Happy watching to your automated deployments.
