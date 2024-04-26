# Simple

[Given configuration](./config.yaml) has:
* listener any pod events in namepsace **app-dev**
* **autoCreate** target elements such as application, category and environment if they do not exist into Golive.

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
  attributes:
    CustomAttribute: Deployment Deploy Attribute Value
  deployeddate: "2024-04-26T15:25:27+02:00"
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

We can see:
* **autoCreate** is requested to create app/cat/environment if not existing into Golive.
* **environment.attributes** has been populated with the value of the annotation **golive.apwide.net/CustomAttribute** located on the Deployment
* **category.name** has been populated by k8s-golive-monitor default annotation **golive.apwide.net/cat** located on the Deployment
* **application.name** comes from docker image because no **application** has been configured
* **deployment.versionname** has been taken from docker image version
* **environment.name** is loaded from Deployment name because no configuration has been provided
* **status** is null because we have not configured any **statusMapping**

