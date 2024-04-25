Build
```shell
helm package .
```

Install
```shell
helm -n default install k8s-golive-monitor ./k8s-monitor-X.X.X.tgz --values values.yaml
```
