# Helm

Some useful commands:

1. Verify deployed release
```
helm list
```

2. Get values used by a release
```
helm get values <release_name> --all
```

2. Get available values of a chart
```
helm show values <chart>
```

3. Upgrade release (using local chart)
```
helm upgrade --install <release_name> <chart_path>
```

4. Upgrade and set custom values
```
helm upgrade --install <release_name> <chart_path> --set key=value
helm upgrade --install <release_name> <chart_path> --values <values_file_path>
```

5. Add repo
```
helm repo add <repo_name> <repo_url>
```

6. Install chart from repo
```
helm install <name> <chart>
```