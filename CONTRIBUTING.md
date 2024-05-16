Build
```shell
helm package .
```

Install
```shell
helm -n default install k8s-golive-agent ./k8s-golive-agent-X.X.X.tgz --values values.yaml
```
