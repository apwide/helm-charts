# Simple Namespace Status

[Given configuration](./config.yaml) has:
* extract **category** information from the **namespace**
* **statusMapping** to map pod status to corresponding Golive status. (can be by name or id)
* **application** name is configured to load its value from a pod's owner (Deployment, StatefulSet, DaemonSet) custom label *my.company.com/label*

Based on the example [app](../app/app.yaml), here is what will be pushed to Golive:

*null values have been removed for clarity*
```yaml
deployment:
    attributes:
        CustomAttribute: Deployment Deploy Attribute Value
    deployeddate: "2024-04-29T08:02:46+02:00"
    versionname: 1.14.2-SNAPSHOT
environment:
    attributes:
        CustomAttribute: Deployment Environment Attribute Value
        Environment Parameter: My Env Variable
        Team: Apwide
    name: nginx - custom-namespace-label // nginx (custom-deployment-label) [custom-deployment-annotation]
environmentselector:
    application:
        autocreate: true
        name: nginx
    category:
        autocreate: true
        name: custom-namespace-label
    environment:
        autocreate: true
        name: nginx - custom-namespace-label // nginx (custom-deployment-label) [custom-deployment-annotation]
```

We can see:
* **environment.attributes** has been completed with:
  * some hard-coded value in the configuration (**Team**)
  * and a **fromPath** (jsonpath) expression evaluated on the deployment
* **deployment.versionname** has been processed with a template expression to append **-SNAPSHOT** to its **.Value** 
* **environment.name** is also generated from an expression using:
  * **.App.Name** name of the application found and present in the template context
  * **.Cat.Name** name of the category found and present in the the template context
  * a **jsonPath** template function evaluated on the deployment
  * a **label** template function to extract label from deployment
  * an **annotation** template function to extract annotation from deployment

