# Simple

[Given configuration](./config.yaml) has:
* listener any pod events in namepsace **app-dev**
* **autoCreate** target elements such as application, category and environment if they do not exist into Golive.

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
  deployeddate: "2024-04-24T14:33:57Z"
  versionname: 1.14.2
environment:
  attributes:
    CustomAttribute: Deployment Attribute Value
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
status: null

```

We can see:
* **autoCreate** is requested to create app/cat/environment if not existing into Golive.
* **environment.attributes** has been populated with the value of the annotation **golive.apwide.net/CustomAttribute** located on the Deployment
* **category.name** has been populated by k8s-golive-monitor default annotation **golive.apwide.net/cat** located on the Deployment
* **application.name** comes from docker image because no **application** has been configured
* **deployment.versionname** has been taken from docker image version
* **environment.name** is loaded from Deployment name because no configuration has been provided
* **status** is null because we have not configured any **statusMapping**

