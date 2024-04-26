# Simple Namespace Status

[Given configuration](./config.yaml) has:
* **statusMapping** to map pod status to corresponding Golive status. (can be by name or id)

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
  attributes:
    CustomAttribute: Deployment Deploy Attribute Value
  deployeddate: "2024-04-26T15:26:13+02:00"
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
status:
  name: Up

```

We can see:
* **status** is not anymore null but mapped to Golive status name corresponding to an up pod
