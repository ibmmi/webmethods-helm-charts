## API GW Setup on Openshift Environment 

Prerequisites

 [ECK / Elasticsearch](https://github.com/ibmmi/webmethods-helm-charts/blob/main/apigateway/helm/README.md#prerequisites)

Technical Steps
    Openshift provides both an administration UI and the oc command line tool. This page describes the steps based on the
    command line tool.

Create a new openshift project
        oc new-project api-gw-01

Create a service account api-gw-sa.
        oc create serviceaccount api-gw-sa 

Assign the permission to the service account api-gw-sa to use the built-in user of the image. ( Note: You must have OpenShift administrator privileges to perform this step)
        oc adm policy add-scc-to-user anyuid -z api-gw-sa

Create regcred credentials inside project

Following Changes are required to be done in values.xml file
        
        serviceAccount:
            create: false
            name: "api-gw-sa"

        elasticsearch:
            storageClassName : "enter storage class name here"
            serviceAccount:
                create: false
                name: "api-gw-sa"
        kibana:
            serviceAccount:
                create: false
                name: "api-gw-sa"

 [Adding webMethod helm repo ](https://github.com/ibmmi/webmethods-helm-charts/blob/main/README.md#helm-chart-repository-for-ibm-webmethods-products-and-components)

