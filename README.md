# ApplicationSet Addons

## Overview

Let's apply Apps of Apps pattern using Argo ApplicationSet.

## Prerequisites

- Kubernetes cluster

```bash
kind create cluster --config kind-1.29.yaml
```

- ArgoCD installed

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Repository structure

Here's a brief description of the main directories and files:

`applications/`: This directory contains the main Helm chart for App of Apps Pattern (v1 approach). The `Chart.yaml` file describes the chart and its version, and the `values.yaml` file specifies configuration values for the chart. The `templates/` subdirectory contains template files that represents Argo Applications.

`bootstrap/`: This directory contains YAML files that are used to bootstrap ApplicationSets (v2 approach). Together with `bootstrap.yaml` file, these files are used to provision ApplicationSets.

`charts/`: This directory contains Helm charts for various add-ons or dependencies. Each subdirectory (like addons/, capsule-system/, ingress-nginx/, etc.) represents a different add-on, and contains its own Chart.yaml and values.yaml files.

`clusters/`: This directory contains configuration specific to different Kubernetes clusters where application might be deployed. The dev/, in-cluster/, and prod/ subdirectories correspond to different environments (development, production, etc.). You can use this directory to override values based on the add-ons+environment combination.

`kind-1.29.yaml`: This file is a configuration file for running a local Kubernetes cluster using Kind.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: addons
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - directory:
      repoURL: https://github.com/sergk/appset-addons
      revision: HEAD
      directories:
      - path: '*/*/*'
  template:
    metadata:
      name: '{{index .path.segments 0}}-{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/sergk/appset-addons
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        name: '{{index .path.segments 0}}'
        namespace: '{{path.basename}}'
```
