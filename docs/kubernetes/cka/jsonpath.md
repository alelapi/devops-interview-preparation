# Jsonpath and custom-columns

## Jsonpath
Print all pods names:
```
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

Loop over multiple objects with `range`
```
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" - "}{.status.phase}{"\n"}{end}'
```

Filter with condition:
```
kubectl get pods -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'
```

## Custom columns
Get name and status in a table like format:
```
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"
```

