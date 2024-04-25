# Simple Namespace Status

[Given configuration](./config.yaml) has:
* extract **category** information from the **namespace**
* **statusMapping** to map pod status to corresponding Golive status. (can be by name or id)
* **application** name is configured to load its value from a pod's owner (Deployment, StatefulSet, DaemonSet) custom label *my.company.com/label*

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
  deployeddate: "2024-04-24T14:44:48Z"
  versionname: 1.14.2
environment:
  attributes:
    CustomAttribute: Deployment Attribute Value
  name: payment-app
environmentselector:
  application:
    autocreate: true
    name: custom-deployment-label
  category:
    autocreate: true
    name: Default Namespace Label
  environment:
    autocreate: true
    name: payment-app
status:
  name: Up

```

We can see:
* **category.name**  comes from docker image because no **application** has been configured
* **application.name** is now populated from deployment custom label **my.company.com/label** value
* **status** is not anymore null but mapped to Golive status name corresponding to an up pod
