# ApplicationSet Addons

## Overview

Let's apply Apps of Apps pattern using Argo ApplicationSet.

Repository structure:

Here's a brief description of the main directories:

`bootstrap/`: This directory contains bootstrap ApplicationSets, e.g. `addons-appset.yaml` which creates ApplicationSet for specific applications.

`charts/`: This directory contains Helm chart directories for various applications and services like capsule-system, cert-manager, ingress-nginx, prometheus-operator, etc. Each of these directories contains a Chart.yaml file which defines the Helm chart, and a values.yaml file which provides default configuration values for the chart.

`clusters/`: This directory contains subdirectories for different environments like dev, in-cluster, and prod. Each of these environment directories contains an `addons/` directory, which is used for environment-specific values of add-ons.

`configs/`: This directory contains configuration files for different environments (dev, in-cluster, prod). It is used to have specific values for ApplicationSet TemplatePatch approach. So we can have different values of Argo Application for different environments.

`bootstrap.yaml` - Provision ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
    - matrix:
        generators:
          - git:
              repoURL: https://github.com/sergk/appset-addons
              revision: poc
              files:
                - path: configs/*/*/config.yaml

          - clusters:
              selector:
                matchExpressions:
                # Check label to see if addon is enabled.
                - key: enable_{{ .path.basename | snakecase }}
                  operator: In
                  values: ['true']
  template:
    metadata:
      name: 'addon-{{.name}}-{{.path.basename}}'
    spec:
      project: default
      source:
        helm:
          releaseName: '{{.path.basename}}'
          ignoreMissingValueFiles: true
          valueFiles:
          - '../../../clusters/{{.name}}/addons/{{.path.basename}}.yaml'
        repoURL: 'https://github.com/sergk/appset-addons'
        path: 'charts/addons/{{.path.basename}}'
        targetRevision: poc
      destination:
        namespace: '{{.path.basename}}'
        name: '{{.name}}'
      syncPolicy:
        syncOptions:
          - CreateNamespace=true
          - ServerSideApply=true  # Big CRDs.
      ignoreDifferences:
          - group: admissionregistration.k8s.io
            kind: MutatingWebhookConfiguration
            jqPathExpressions:
            - .webhooks[].clientConfig.caBundle
          - group: admissionregistration.k8s.io
            kind: ValidatingWebhookConfiguration
            jqPathExpressions:
            - .webhooks[].clientConfig.caBundle

  templatePatch: |
    {{- if .autoSync }}
    spec:
      syncPolicy:
        automated:
          prune: {{ .prune }}
    {{- end }}
```
