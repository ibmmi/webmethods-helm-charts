# API Gateway Setup on Openshift Environment 

## Prerequisites

[ECK / Elasticsearch](https://github.com/ibmmi/webmethods-helm-charts/blob/main/apigateway/helm/README.md#prerequisites)  Install ECK Operator to deploy elasticsearch.

## Technical Steps

Openshift provides oc command line tool. This page describes the steps based on the command line tool.

## Create a new openshift project

```sh
oc new-project api-gw-01
```

## Create a service account api-gw-sa

```sh
oc create serviceaccount api-gw-sa 
```

## Image Pull secrets

Setup Container registry image credentials to download images inside the project. [wM Containers](https://containers.webmethods.io/)

```sh
kubectl create secret \

    docker-registry regcred \
    --docker-server=<server-url> \
    --docker-username=<user-name> \
    --docker-password=<password> \
    --docker-email=<email> \
    -n api-gw-01
```

## Following Changes are required to be done in values.xml file

```yaml
serviceAccount:
    create: false
    name: "api-gw-sa-02"

elasticsearch:
    storageClassName : "enter storage class name"
    serviceAccount:
        create: false
        name: "api-gw-sa-02"
kibana:
    serviceAccount:
        create: false
        name: "api-gw-sa-02"
```

## Setup Helm repo for webMethods

[webMethods Helm Repo](https://github.com/ibmmi/webmethods-helm-charts/blob/main/README.md#helm-chart-repository-for-ibm-webmethods-products-and-components)

```sh
helm repo add webmethods https://ibm.github.io/webmethods-helm-charts/charts
```

```sh
helm repo add webmethods ${REPO_HOME}
```

## Install helm release

Shell

```sh
helm install api-gw-01 webmethods/apigateway \
    -f mysetup-values.yaml \
    --set-file license=licenseKey.xml
```

Windows commandline

```bat
helm install api-gw-01 webmethods/apigateway `
    -f mysetup-values.yaml `
    --set-file license=licenseKey.xml
```

```bat
helm install api-gw-02 `
.\apigateway\helm\ `
-f .\apigateway\openshift\techzone-values.yaml `
--set-file license=.\\apigateway\\openshift\\licenseKey.xml
```

Please provide API Gateway License file.

## Creating routes using following commands

```sh
oc expose svc/api-gw-01-apigateway-ui
oc expose svc/api-gw-01-apigateway-admin
```

```sh
export RELEASE_NAME=api-gw-02
oc expose svc/${RELEASE_NAME}-apigateway-ui
oc expose svc/${RELEASE_NAME}-apigateway-admin
```
