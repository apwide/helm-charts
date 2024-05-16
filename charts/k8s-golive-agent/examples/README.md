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
kubectl -n k8s-golive-agent rollout status deployment k8s-golive-agent
kubectl -n k8s-golive-agent logs -l app=k8s-golive-agent --tail=50 -f
```

## Status
Let's share the status of your environments with Golive & Jira users.

```shell
kubectl apply -k ./status
kubectl -n k8s-golive-agent rollout status deployment k8s-golive-agent
kubectl -n k8s-golive-agent logs -l app=k8s-golive-agent --tail=50 -f
```

## Templating
To give you more flexibility in how to extract application, category, version, and environment information,
let's explore some examples of advanced templating that can be used to extract information.

```shell
kubectl apply -k ./templating
kubectl -n k8s-golive-agent rollout status deployment k8s-golive-agent
kubectl -n k8s-golive-agent logs -l app=k8s-golive-agent --tail=50 -f
```

## Selectors
Here, we will explore selectors that can be useful for restricting the subset of environments
you want to keep track of, or if you want to apply specific data extraction policies to specific resources.

```shell
kubectl apply -k ./selectors
kubectl -n k8s-golive-agent rollout status deployment k8s-golive-agent
kubectl -n k8s-golive-agent logs -l app=k8s-golive-agent --tail=50 -f
```

