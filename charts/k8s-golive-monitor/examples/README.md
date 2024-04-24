# Examples

You can apply the following example with [kubectl kustomize](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_kustomize/)
on your cluster:

When you're done:
```shell
kubectl delete -k ./simple
```

## Simple
One of the simplest configurations is to delegate most of the data extraction to the default policies of the operator.

```shell
kubectl apply -k ./simple
kubectl -n k8s-golive-monitor rollout status deployment k8s-golive-monitor
kubectl -n k8s-golive-monitor logs -l app=k8s-golive-monitor --tail=50 -f
```

## Simple Namespace Status
Let's share the status of your environments with Golive & Jira users.
Since it's common to use namespaces to categorize environments, let's extract information from them.

```shell
kubectl apply -k ./simple-namespace-status
kubectl -n k8s-golive-monitor rollout status deployment k8s-golive-monitor
kubectl -n k8s-golive-monitor logs -l app=k8s-golive-monitor --tail=50 -f
```

## Advanced Expressions
To give you more flexibility in naming your application, category, version, and environment,
let's explore some examples of advanced expressions that can be used to extract information.

```shell
kubectl apply -k ./advanced-expressions
kubectl -n k8s-golive-monitor rollout status deployment k8s-golive-monitor
kubectl -n k8s-golive-monitor logs -l app=k8s-golive-monitor --tail=50 -f
```

## Selectors
Here, we will explore selectors that can be useful for restricting the subset of environments
you want to keep track of, or if you want to apply specific data extraction policies to specific resources.

```shell
kubectl apply -k ./selectors
kubectl -n k8s-golive-monitor rollout status deployment k8s-golive-monitor
kubectl -n k8s-golive-monitor logs -l app=k8s-golive-monitor --tail=50 -f
```

