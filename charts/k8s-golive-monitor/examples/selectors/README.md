# Simple Namespace Status

[Given configuration](./config.yaml) has:
* no more **namespace** selector, so, all namespaces will be monitored.
* 2 **labels** key/value selectors (app and )
* a **labelQuery** selector which is equivalent to our first key/value selector but using *set-based* selector.

Selectors are applied on pod. Pod must satisfy each of the criteria (namespace, label, labelQuery) specified in a given selector,
but does not have to match all of the selector in case several are configured.

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
  attributes:
    CustomAttribute: Deployment Deploy Attribute Value
  deployeddate: "2024-04-26T15:27:50+02:00"
  versionname: 1.14.2
environment:
  attributes:
    CustomAttribute: Deployment Environment Attribute Value
  name: payment-app
environmentselector:
  application:
    autocreate: true
    name: nginx
  category:
    autocreate: true
    name: Default Deployment Annotation
  environment:
    autocreate: true
    name: payment-app
```

We can see pod has correctly been selected by our selectors.

