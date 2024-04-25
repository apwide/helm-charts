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

Learn more about Apwide Golive: [https://www.apwide.com](https://www.apwide.com)

# How it works

The controller listens for POD events and, based on configured selectors, determines if a POD falls within the scope of monitored resources.
If so, the controller identifies the resource owning the POD (e.g., Deployment, StatefulSet, DaemonSet) and applies various data extraction strategies
to compile environment information, subsequently sending it to Golive.

# Usage

## Add Repository
```shell
helm repo add apwide https://apwide.github.io/helm-charts
helm repo update
```
## Install Helm Chart
```shell
helm install [RELEASE_NAME] apwide/k8s-golive-monitr
```

# Configuration

Show available options for this chart:
````shell
helm show values apwide/k8s-golive-monitor
````

# Examples

Have a look at our [various examples](https://github.com/apwide/helm-charts/tree/main/charts/k8s-golive-monitor/examples) and publish them on your cluster to see how k8s-golive-monitor can be configured.

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
* **name**: how to extract/select the environment name.
* **attributes**: what and how to extract environment attribute values.
* **version**: how to extract the deployment version name.
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

## Data Extractions

k8s-golive-monitor listens to pod events and apply **selectors** on it to know if a pod should be monitored.
However, to extract data, it focuses on the pod owner resource: Deployment, StatefulSet, DaemonSet.
Only these three are currently supported (Jobs, single Pods, etc., are ignored).

When a pod matches a selector, k8s-golive-monitor finds the owning resource
and applies the extraction rules to it to retrieve application, category, version, etc.

The following sections describes how to extract environment information.

## Category

Category section is to extract the environment category name:
```yaml
listeners:
  - id: my-listener
    category:
      namespace: true
      value: "Dev"
      label: "my.company.com/cat"
      annotation: "my.company.com/env"
      template: |
        {{ .Value }} ({{ label "my.company.com/cat"  }})
```

### Parameters
* **namespace** : a boolean specifying if we want to search for a label/annotation/template at the namespace level or the owner level (e.g., Deployment).
* **value** : a hard-coded value taken as is from the configuration.
* **label** : name taken from the value of this label.
* **annotation** : value of this annotation is used.
* **template** : possibility to customize the value with [template expression](#template-expression)

For the template expression, here is the available context element:
* **Value** : value matched by the previous rule


### Precedence
In case a parameter is specified and doesn't match, an error is raised and data are not sent.

Precedence order to get category name when namespace=true
1. **value** from config
2. **label** namespace
3. **annotation** from namespace
4. **golive.apwide.net/cat** default label on namespace
5. **golive.apwide.net/cat** default annotation on namespace
6. **namespace name** in last resort

Precedence order to get category name when namespace=false
1. **value** from config
2. **label** on owning resource
3. **annotation** on owning resource
4. **golive.apwide.net/cat** default label on owning resource
5. **golive.apwide.net/cat** default annotation on owning resource

## Application

Application section is to extract Golive application name:
```yaml
listeners:
  - id: my-listener
    application:
      namespace: true
      value: "Payment"
      label: "my.company.com/app"
      annotation: "my.company.com/env-name"
      template: |
        {{ .Value }} ({{ label "my.company.com/app"  }})
```

### Parameters
* **namespace** : a boolean specifying if we want to search for a label/annotation/template at the namespace level or the owner level (e.g., Deployment).
* **value** : a hard-coded value taken as is from the configuration.
* **label** : name taken from the value of this label.
* **annotation** : value of this annotation is used.
* **template** : possibility to customize the value with [template expression](#template-expression)

For the template expression, here is the available context element:
* **Value** : value matched by the previous rule

### Precedence
In case a parameter is specified and doesn't match, an error is raised and data are not sent.

Precedence order to get application name when namespace=true
1. **value** from config
2. **label** namespace
3. **annotation** from namespace
4. **golive.apwide.net/app** default label on namespace
5. **golive.apwide.net/app** default annotation on namespace
6. **namespace name** in last resort

Precedence order to get application name when namespace=false
1. **value** from config
2. **label** on owning resource
3. **annotation** on owning resource
4. **golive.apwide.net/app** default label on owning resource
5. **golive.apwide.net/app** default annotation on owning resource
6. **docker image name** in last resort

## Version

Version section is to extract Golive deployment version name:
```yaml
listeners:
  - id: my-listener
    version:
      ignore: false
      namespace: true
      value: "1.0"
      label: "my.company.com/app"
      annotation: "my.company.com/env-name"
      template: |
        {{ .Value }}-SNAPSHOT
```

### Parameters
* **ignore** : flag to ignore tracking of version/deployment information.
* **namespace** : a boolean specifying if we want to search for a label/annotation/template at the namespace level or the owner level (e.g., Deployment).
* **value** : a hard-coded value taken as is from the configuration.
* **label** : name taken from the value of this label.
* **annotation** : value of this annotation is used.
* **template** : possibility to customize the value with [template expression](#template-expression)

For the template expression, here is the available context element:
* **Value** : value matched by the previous rule

### Precedence
In case a parameter is specified and doesn't match, an error is raised and data are not sent.

Precedence order to get version name when namespace=true
1. **value** from config
2. **label** namespace
3. **annotation** from namespace
4. **golive.apwide.net/app** default label on namespace
5. **golive.apwide.net/app** default annotation on namespace
6. **namespace name** in last resort

Precedence order to get application name when namespace=false
1. **value** from config
2. **label** on owning resource
3. **annotation** on owning resource
4. **golive.apwide.net/app** default label on owning resource
5. **golive.apwide.net/app** default annotation on owning resource
6. **docker image version** in last resort

## Name
Name section is to extract Golive environment name:
```yaml
listeners:
  - id: my-listener
    name:
      namespace: true
      value: "1.0"
      label: "my.company.com/app"
      annotation: "my.company.com/env-name"
      template: |
        {{ .Value }}-SNAPSHOT
```

### Parameters
* **namespace** : boolean to specify if we want to search for label/annotation/template at namespace level or owner (eg: Deployment)
* **value** : hard-coded value taken as is from the configuration
* **label** : name taken from value of this label
* **annotation** : value of this annotation used
* **template** : possibility to customize the value with [template expression](#template-expression)

For the template expression, here is the available context element:
* **Value** : value matched by the previous rule
* **App.Name** : application name as processed by previous section
* **Cat.Name** : category name as processed by previous section

### Precedence
In case a parameter is specified and doesn't match, an error is raised and data are not sent.

Precedence order to get environment name when namespace=true
1. **value** from config
2. **label** namespace
3. **annotation** from namespace
4. **golive.apwide.net/app** default label on namespace
5. **golive.apwide.net/app** default annotation on namespace
6. **namespace name**

Precedence order to get environment name when namespace=false
1. **value** from config
2. **label** on owning resource
3. **annotation** on owning resource
4. **golive.apwide.net/app** default label on owning resource
5. **golive.apwide.net/app** default annotation on owning resource
6. **owning resource name** in last resort

## Environment Attributes
An array of expression to extract environment attributes:
```yaml
listeners:
  - id: my-listener
    environmentAttributes:
      - name: Team
        value: Apwide
      - name: Environment Variable
        fromPath: .spec.template.spec.containers[0].env[?(@.name == 'MY_ENV_VARIABLE')].value
```

### Parameters

* **name**: name of the attribute. (attribute must exists into Golive)
* **value**: configuration hard-coded value for the given attribute.
* **fromPath**: json path expression evaluated on owning resource

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
