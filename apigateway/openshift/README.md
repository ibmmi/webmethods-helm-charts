# API Gateway Setup on Openshift Environment 

 ## Prerequisites
    
[ECK / Elasticsearch](https://github.com/ibmmi/webmethods-helm-charts/blob/main/apigateway/helm/README.md#prerequisites)  Install ECK Operator to deploy elasticsearch.

## Technical Steps
Openshift provides oc command line tool. This page describes the steps based on the command line tool.

## Create a new openshift project
    oc new-project api-gw-01

## Create a service account api-gw-sa.
    oc create serviceaccount api-gw-sa 

## Service account permission
Assign the permission to the service account api-gw-sa to use the built-in user of the image. ( Note: You must have OpenShift administrator privileges to perform this step)
        
    oc adm policy add-scc-to-user anyuid -z api-gw-sa

## Image Pull secrets
Setup Container registry image credentials to download images inside the project. [wM Containers](https://containers.webmethods.io/)

## Following Changes are required to be done in values.xml file
        
        serviceAccount:
            create: false
            name: "api-gw-sa"

        elasticsearch:
            storageClassName : "enter storage class name"
            serviceAccount:
                create: false
                name: "api-gw-sa"
        kibana:
            serviceAccount:
                create: false
                name: "api-gw-sa"

## Setup Helm repo for webMethods.
[webMethods Helm Repo](https://github.com/ibmmi/webmethods-helm-charts/blob/main/README.md#helm-chart-repository-for-ibm-webmethods-products-and-components)

## Install helm release 
  helm install api-gw-01 webmethods/apigateway -f mysetup-values.yaml --set-file license=licenseKey.xml
Please provide API Gateway License file.

## Creating routes using following commands
   oc expose svc/api-gw-01-apigateway-ui
   oc expose svc/api-gw-01-apigateway-admin 