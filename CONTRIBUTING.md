# Contributing

1. Bump the chart `version` in `Chart.yaml` for any change under `charts/<name>`.
2. Document new values with a `# --` comment and run `helm-docs`.
3. Add or update a test in `charts/<name>/tests` and, for new integrations, a scenario in `charts/<name>/ci`.
4. Run locally before opening a pull request:

```console
helm lint --strict charts/app
helm unittest charts/app
for f in charts/app/ci/*.yaml; do helm template t charts/app -f "$f" | kubeconform -strict -ignore-missing-schemas; done
```
