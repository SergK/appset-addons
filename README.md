# ApplicationSet Addons

## Overview

Let's apply Apps of Apps pattern using Argo ApplicationSet.

Repository structure:

bootstrap.yaml - Provision ApplicationSet


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
