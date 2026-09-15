# Helm Interview Questions and Answers

## 1. What is Helm?
Helm is a package manager for Kubernetes. It packages Kubernetes manifests into reusable charts and manages releases.

## 2. What is a Helm Chart?
A Chart is a collection of files describing a Kubernetes application.

Typical structure:
```text
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
└── charts/
```

## 3. What is `Chart.yaml`?
It contains chart metadata such as:
- Name
- Version
- Description
- Application version
- Dependencies

## 4. What is `values.yaml`?
It contains default configuration values used by chart templates.

## 5. What are Helm templates?
Templates are Kubernetes manifest files containing Go-template expressions and Helm functions.

## 6. What is a Helm release?
A release is an installed instance of a chart in a Kubernetes Namespace.

The same chart can be installed multiple times with different release names and values.

## 7. What is the difference between chart and release?
- Chart = installable package.
- Release = deployed instance of that package.

## 8. How do you install a chart?
```bash
helm install myapp ./mychart
```

## 9. How do you install with custom values?
```bash
helm install myapp ./mychart \
  -f values-prod.yaml \
  --set image.tag=v2
```

## 10. How do you upgrade a release?
```bash
helm upgrade myapp ./mychart
```

## 11. How do you roll back?
```bash
helm rollback myapp 1
```

## 12. How do you inspect releases?
```bash
helm list -A
helm status myapp
helm history myapp
```

## 13. What is `helm template`?
It renders templates locally without installing them.

```bash
helm template myapp ./mychart
```

## 14. What is `helm lint`?
It checks a chart for possible structural and formatting problems.

```bash
helm lint ./mychart
```

## 15. What is `helm upgrade --install`?
It installs a release if it does not exist, otherwise upgrades it.

```bash
helm upgrade --install myapp ./mychart
```

## 16. What is a Helm repository?
A repository stores and distributes packaged charts and repository metadata.

## 17. What are chart dependencies?
Dependencies are other charts required by a chart.

They can be declared in `Chart.yaml`.

## 18. What is the difference between Helm v2 and Helm v3?
Helm v3 removed Tiller. Helm v3 uses the Kubernetes API and stores release information in the cluster, commonly as Secrets.

## 19. What is a named template?
A reusable template fragment defined with `define` and called with `include` or `template`.

## 20. What is the difference between `include` and `template`?
`include` returns rendered content as a string and can be passed through functions such as `indent`. `template` directly renders the named template.

## 21. What is `required`?
`required` causes rendering to fail if a required value is missing.

```gotemplate
{{ required "image.repository is required" .Values.image.repository }}
```

## 22. What is `default`?
`default` provides a fallback value when a value is empty.

## 23. What is `lookup`?
`lookup` queries existing Kubernetes resources during rendering when supported by the execution context.

## 24. What is a Helm hook?
A hook allows a resource or Job to run at a lifecycle point such as pre-install, post-install, pre-upgrade, or post-delete.

## 25. What is `helm diff`?
`helm diff` is a commonly used plugin that shows differences between the current release and a proposed change.

## 26. How do you debug Helm rendering?
```bash
helm lint ./mychart
helm template myapp ./mychart --debug
helm install myapp ./mychart --dry-run --debug
```

## 27. What is the difference between `--set` and `-f`?
- `--set` overrides individual values from the command line.
- `-f` supplies a values file.
- Multiple values files are merged in order, with later values taking precedence.

## 28. What are Helm best practices?
- Keep charts modular.
- Use meaningful values.
- Validate with `helm lint`.
- Render before deployment.
- Pin chart and image versions.
- Avoid hardcoding environment-specific values.
- Use semantic chart versions.
- Store charts in version control.
- Use CI to validate templates.
- Be careful with hooks and destructive upgrades.
