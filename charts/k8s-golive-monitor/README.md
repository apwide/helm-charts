# k8s-golive-monitor

k8s-golive-monitor is a Kubernetes controller designed to manage environment information within your Kubernetes cluster,
including deployed version tracking and the statuses of deployments, stateful sets, and daemon sets.
It facilitates the publication of this data to Apwide Golive for Jira.

# What is Apwide Golive for Jira?

Apwide Golive is the game-changing solution for comprehensive test environment management.

Seamlessly integrated with your development and release processes, Apwide Golive empowers your teams to
deliver high-quality software with speed and confidence.
Apwide Golive provides a centralized dashboard for visualizing and tracking environment usage, resource allocation, and availability right in Jira.

With integrated notifications and approvals in Jira, stakeholders stay informed and can provide quick feedback, reducing delays and accelerating the testing
process.

Learn more about Apwide Golive: https://www.apwide.com

# How it works

The controller listens for POD events and, based on configured selectors, determines if a POD falls within the scope of monitored resources.
If so, the controller identifies the resource owning the POD (e.g., Deployment, StatefulSet, DaemonSet) and applies various data extraction strategies
to compile environment information, subsequently sending it to Golive.

# Examples

Have a look at our [various examples](https://github.com/apwide/helm-charts/tree/main/charts/k8s-golive-monitor/examples) and publish them on your cluster to see how it works

# Usage from Helm Chart
Add repo:
```shell
helm repo add apwide https://apwide.github.io/helm-charts
helm repo update
```

Install helm
## Install Helm Chart
```shell
helm install [RELEASE_NAME] apwide/k8s-golive-monitr
```


Configure values.
By default, the controller runs in offline mode, logging the data that would be sent to Golive. This facilitates initial configuration.

**_Make sure you have selected what you want to track and what data need to be sent before connecting the controller to your Golive instance._**

````shell
helm show values apwide/k8s-golive-monitor
````

Connect controller to your Golive by filling helm auth values:
```yaml
golive:
  auth:
    token: "XXXX" # Golive API Token
  config:
    golive:
      offline: false
```

# Configuration

## Golive

By default, the controller runs in offline mode, logging the data that would be sent to Golive. This facilitates initial configuration.

The most basic configuration includes:
```yaml
golive:
  offline: true
listeners:
  - id: my-listener
```
At least one listener is mandatory.

Offline mode supports logging Golive payload in YAML for improved readability in console logs:
```yaml
golive:
  offline: true
  yaml: true
```

When ready, offline mode can be disabled, and the Golive API client can be configured to push information.

**Golive Cloud**: you only need an [authentication token](https://www.apwide.com/golive/cloud/environments/help/how-to-use-api-tokens)
```yaml
golive:
  offline: false
  token: "eyJra[...]xKWvNoQ"
```

**Golive Server**: you need to specify the Golive base API url for your given Jira instance and its credentials:
```yaml
golive:
  offline: false
  # https://[jiraBaseUrl]:[jiraPort]/[jiraContextPath]/rest/apwide/tem/1.1
  url: https://jira.company.com/rest/apwide/tem/1.1
  username: golive-service-account
  password: XXXX
```

## Listeners

Listeners use **selectors** to specify what to track and have configurations on how to extract environment/deployment data to push them to Golive.

A listener consists of:

* **id**: a unique identifier used in logs to identify the configuration used.
* **autoCreate**: a boolean specifying if targets (application, category, and environment) must be created if they do not exist in Golive and if the call fails.
* **category**: how to extract/select the category.
* **application**: how to extract/select the application.
* **environment**: how to extract/select the environment information (eg: attributes, url, name...).
* **deployment**: how to extract the deployment version name.
* **selectors**: pod selectors used to determine which events match this configuration.

## Selectors

Selectors are applied to pods to determine if they are monitored:
```yaml
listeners:
  - id: frontend-monitoring
    selectors:
      - namespace: app-dev
        labels:
          app: dev
        labelQuery: app in (dev)
```

Three types of selectors are currently supported:

* **namespace**: restricts to pods in the given namespace.
* **labels**: a key/value map that must match a pod (logical AND).
* **labelQuery**: a [set-based expression](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement) that pod labels must match.

A pod must satisfy each criterion (namespace, label, labelQuery) specified in a given selector. However, a listener can have several selectors,
and a pod must match all the criteria of only one of them. If a pod matches multiple selectors in different listeners, the first declared listener wins.

## Data Identification and Extraction

k8s-golive-monitor listens to pod events and apply **selectors** on it to know if a pod should be monitored.
However, to extract data, it focuses on the pod owner resource: Deployment, StatefulSet, DaemonSet.
Only these three are currently supported (Jobs, single Pods, etc., are ignored).

When a pod matches a selector, k8s-golive-monitor finds the owning resource
and applies the extraction rules to it to retrieve application, category, version, etc.

The following sections describes how to extract environment information.

## Simple Example

The following example listens for pod events in the **dev** namespace and pushes predefined values for each environment attribute.
If the application, category, and environment do not exist in Golive, they will be automatically created (**autoCreate**).

```yaml
listeners:
  - id: hard-coded-example
    selectors:
      - namespace: dev
        labelQuery: "app in (payment-dev)"
    autoCreate: true
    category:
      name: Dev
    application:
      name: Payment
    deployment:
      versionName: latest
      attributes:
        - name: Monitored By
          value: k8s-golive-monitor
    environment:
      name: Payment Dev
      url: http://my.company.com/payment
      attributes:
        - name: Team
          value: My Company
```

While this example is straightforward, hard-coding data for every environment we wish to monitor isn't particularly practical.
Let's explore the next example, which captures dynamic information.

*For a more comprehensive understanding of the environment data model, refer to the [Golive documentation](https://golive.apwide.com/doc/latest/cloud/understand-environment-core-concepts)*.

## Templating

k8s-golive-monitor employs [golang template](https://pkg.go.dev/text/template) (e.g., {{ .myExpression }})
to define values for various sections within our environment.

Here's another example demonstrating the use of templating:
```yaml
listeners:
  - id: template-example
    selectors:
      - namespace: dev
    autoCreate: true
    category:
      name: "{{ nsName }}"
    application:
      name: "{{ nsDefaultAnnotation }}"
    deployment:
      versionName: "{{ mainImageTag }}-SNAPSHOT"
      attributes:
        - name: Monitored By
          value: '{{ nsLabel "my.company.com/monitored-by" }}'
    environment:
      name: "{{ .App.Name }} - {{ .Cat.Name }}"
      url: |
        {{ jsonPath ".spec.template.spec.containers[0].env[?(@.name=='BASE_URL')].value" }}
      attributes:
        - name: Team
          value: '{{ annotation "my.company.com/owners" }}'
```

This becomes more beneficial. We've updated the selectors to listen for any pod events in the dev namespace.
For each event, we dynamically map information to our Golive environment:
* **category.name**: we load the category name from the namespace name.
* **application.name**: application name comes from the default annotation, which is for the application *golive.apwide.net/app*
* **deployment.versionName**: deployment version name comes from the first container image tag with the suffix *-SNAPSHOT*
* Deployment attributes **Monitored By**: the value of the attribute is taken from the namespace label *my.company.com/monitored-by*
* **environment.name**: this is the concatenation of the values previously found for application name and category name.
* **environment.url**: comes from a json path evaluated on the pod owner template (eg: Deployment) which returns the value environment variable named *BASE_URL*
* Environment attributes **Team**: select the value of annotation *my.company.com/owners*

We can observe that we can now retrieve various environment information using a single template.
To streamline access to this information, templates have been enriched with various [utility functions](#template-expression), such as **jsonPath**,
which are detailed below.

Each function is assessed on the resource owning a pod, such as Deployment, StatefulSet, or DaemonSet.
Additionally, there are equivalent functions prefixed with **ns** to be assessed on the namespace (e.g. **nsJsonPath**).

Currently, standalone pods and Jobs are not supported, and events related to them will be disregarded.

Some of these functions make reference to **defaultLabel** or **defaultAnnotation**,
which are special [Golive label/annotation](#default-annotations) that you can add to your resources.
These labels/annotations will be automatically processed as described below.

### Default Values

For certain environment information, when no configuration is set,
the controller attempts to infer the value autonomously by applying a series of templates successfully:

#### Application Name

1. **{{ defaultLabel }}** : value of *golive.apwide.net/app* on the resource
2. **{{ defaultAnnotation }}**: value of *golive.apwide.net/app* on the resource
3. **{{ mainImageName }}**: image name extracted from the first container image

#### Category Name

1. **{{ defaultLabel }}** : value of *golive.apwide.net/cat* on the resource
2. **{{ defaultAnnotation }}**: value of *golive.apwide.net/cat* on the resource
3. **{{ nsDefaultLabel }}** : value of *golive.apwide.net/cat* on the namespace
4. **{{ nsDefaultAnnotation }}**: value of *golive.apwide.net/cat* on the namespace
5. **{{ nsName }}**: name of the namespace

#### Environment Name

1. **{{ defaultLabel }}** : value of *golive.apwide.net/environment* on the resource
2. **{{ defaultAnnotation }}**: value of *golive.apwide.net/environment* on the resource
3. **{{ nsDefaultLabel }}** : value of *golive.apwide.net/environment* on the namespace
4. **{{ nsDefaultAnnotation }}**: value of *golive.apwide.net/environment* on the namespace
5. **{{ name }}**: name of the resource

#### Environment Url

1. **{{ defaultLabel }}** : value of *golive.apwide.net/url* on the resource
2. **{{ defaultAnnotation }}**: value of *golive.apwide.net/url* on the resource
3. **{{ nsDefaultLabel }}** : value of *golive.apwide.net/url* on the namespace
4. **{{ nsDefaultAnnotation }}**: value of *golive.apwide.net/url* on the namespace

#### Deployment Version Name

1. **{{ defaultLabel }}** : value of *golive.apwide.net/version* on the resource
2. **{{ defaultAnnotation }}**: value of *golive.apwide.net/version* on the resource
3. **{{ mainImageTag }}**: image tag extracted from the first container image

## Status
To track environment statuses, you must map operator statuses to Golive statuses.

The operator evaluates the state of an environment using four different statuses:
* down: environment is not running (eg: deployment replicas set to 0)
* up: up and running (eg: deployment desired and read replicas are equals)
* deploy: a change is ongoing (eg: deployment is starting up or a new deployment update happens and pods are restarting)
* failed: deployment failed (eg: deployment has been updated, but pod are not able to start)

Golive statuses can be identified by their ID and/or name:
```yaml
statusMapping:
  down:
    name: Down
  up:
    name: Up
  deploy:
    name: Deploy
  failed:
    name: Down
```

## Template Expression

In various parts of the configuration, [Go template expression](https://pkg.go.dev/text/template) can be used to transform values before sending them to Golive.

In addition to the standard Go template functions, the following additional ones are provided:
* **title** : https://pkg.go.dev/strings#ToTitle
* **lower** : https://pkg.go.dev/strings#ToLower
* **upper** : https://pkg.go.dev/strings#ToUpper
* **annotation** : read annotation value from the owned resource
* **label** : read label value from the owned resource
* **jsonPath** : evaluate the jsonPath expression on the owned resource
* **mainImageName** : docker image name of first container
* **mainImageTag** : docker image version of first container
* **defaultLabel** : search for golive.apwide.net/XXX label where XXX depends on type of data search for [(eg: app, cat, name...)](#default-annotations)
* **defaultAnnotation** : search for golive.apwide.net/XXX annotation where XXX depends on type of data search for [(eg: app, cat, name...)](#default-annotations)
* **nsDefaultLabel** : default label on namespace [(eg: app, cat, name...)](#default-annotations)
* **nsDefaultAnnotation** : default annotation on namespace [(eg: app, cat, name...)](#default-annotations)
* **nsJsonPath** : jsonPath evaluated on namespace
* **nsName** : name of the namespace
* **name** : name of the owning resource


## Default annotations

Here is a set of default annotations which can be set on pod owner resource or namespace which are used in case no template value is provided or
if they are requested by specific template functions: (eg: defaultLabel, defaultAnnotation, nsDefaultLabel, nsDefaultAnnotation)
* golive.apwide.net/app : application
* golive.apwide.net/cat : category
* golive.apwide.net/environment : environment name
* golive.apwide.net/url: environment url
* golive.apwide.net/version: version name
* env.golive.apwide.net/XXX : environment attribute where XXX is the name of the attribute
* deploy.golive.apiwde.net/XXX : deployment attribute where XXX is the name of the attribute

## Environment Variables

Configuration is done using a YAML file. All values can be overridden by environment variables. For example:
```yaml
golive:
  offline: true
```
can be overridden with:
```shell
GOLIVE_OFFLINE=true
```

# Limitations

## Monitored Resources

The operator listens to pods for events but captures information/status from the perspective of their owner.
This means pods created manually without an owner are ignored.

Currently, ownership resource types supported are DaemonSet, StatefulSet, and Deployment.

## Deployment Date

Currently, *Deployment Date* is set to now, so if you have enabled the option "Track re-deployments of the same version",
each time Golive is updated by the operator, a new deployment will be generated.

## Attributes

If you are populating attributes, you must ensure they exist in Golive and create them beforehand.
