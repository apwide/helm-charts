# k8s-golive-monitor

Helm chart install k8s-golive-monitor operator which keep track of your environments in your K8S cluster and push
information to Apwide Golive.

## What is Apwide Golive for Jira?

Apwide Golive is the game-changing solution for comprehensive test environment management.

Seamlessly integrated with your development and release processes, Apwide Golive empowers your teams to
deliver high-quality software with speed and confidence.
Apwide Golive provides a centralized dashboard for visualizing and tracking environment usage, resource allocation, and availability right in Jira.

With integrated notifications and approvals in Jira, stakeholders stay informed and can provide quick feedback, reducing delays and accelerating the testing
process.

Learn more about Apwide Golive: https://www.apwide.com

## Usage

### Add Repository
```shell
helm repo add apwide https://apwide.github.io/helm-charts
helm repo update
```
### Install Helm Chart
```shell
helm install [RELEASE_NAME] apwide/k8s-golive-monitr
```

## Configuration

Show available options for this chart:
````shell
helm show values apwide/k8s-golive-monitor
````
