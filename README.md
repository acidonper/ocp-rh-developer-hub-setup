# Openshift Developer Hub KickStart

The idea of this repository is to provide the required information to onboard quickly in Red Hat Developer Hub understanding the different pieces included in this solution.

There is included a use case *Onboard Jump App Application* at the end of this document in order to understand all pieces working together and the configuration workflows.

## Prerequisites

* Openshift 4.14+

## Developer Hub Setup

In order to install Red Hat Developer Hub, it is required to follow the next steps:

* Create a project in the OpenShift
* Switch to Developer mode on your Red Hat OpenShift web console
* Add -> Helm Chart -> Red Hat Developer Hub -> Create
* Modify variable "OpenShift router host"
* Create

Please visit the following [link](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#proc-install-rhdh-helm_admin-rhdh) for more information about the installation process.

## Install Dynamic Plugins by Red Hat

Red Hat Developer Hub, based on Backstage, is a single-page application composed of a set of plugins.

Red Hat has developed some plugins in order to support and allow multiple integration with third party solutions, for example Red Hat GitOps based on Argocd or Red Hat Pipelines based on Tekton.

There are two main groups of dynamic plugins:

* Frontend plugins -> Frontend components that will be used to display specific information in the client's browser.
* Backend plugins -> The backend uses plugins to construct a working backend that the frontend (app) can use to interact with third party systems and/or make specific actions.

Please review the following [link](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#rhdh-supported-plugins) for more information about dynamic plugins supported by Red Hat.

### Find your plugin

First of all, it is important to find information about the plugin once you have decided to use it. For this mission, you can look for information to [NPM Official Web][https://www.npmjs.com/] using the plugin name (E.g. @roadiehq/scaffolder-backend-argocd).

Once you are in the NPM documentation, it is possible to find the GitHub repository where it is easy to find more information about the plugin.

NOTE: It is important to identify if the respective plugin requires specific env vars and the specific configuration that will be required in the following section

### Configure the Dynamic Plugins

Once the dynamic plugins have been found, it is time to install and configure them following the next steps:

* Add sensitive information via secrets (E.g. ArgoCD information for plugins)

```$bash
oc create -f examples/secret.yaml
```

* Enable the different plugins and add sensitive information (required Env Vars) from the respective secrets

```$yaml
...
global:
    dynamic:
        plugins:     
        - package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd
          disabled: false
        - package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd-backend-dynamic
          disabled: false
        - package: ./dynamic-plugins/dist/roadiehq-scaffolder-backend-argocd-dynamic
          disabled: false
...
upstream:
    backstage:
        appConfig:
            argocd:
                appLocatorMethods:
                    - type: 'config'
                      instances:
                        - name: main
                          url: '${ARGOCD_INSTANCE1_URL}'
                          username: '${ARGOCD_USERNAME}'
                          password: '${ARGOCD_PASSWORD}'
...
        extraEnvVars:
          - name: ARGOCD_INSTANCE1_URL
            valueFrom:
                secretKeyRef:
                    key: url
                    name: argocd-basic-auth
          - name: ARGOCD_USERNAME
            valueFrom:
                secretKeyRef:
                    key: username
                    name: argocd-basic-auth
          - name: ARGOCD_PASSWORD
            valueFrom:
                secretKeyRef:
                    key: password
                    name: argocd-basic-auth
```

### Use Dynamic Plugins

Once the Red Hat Developer Hub pod has been rolled out because of the new configuration inclusion, it is time to use the respective plugin.

It is important to understand the way of interacting with the plugin. For example, it is a plugin that displays specific information in the Red Hat Developer Hub Portal, it maybe does not required extra steps to start playing with it.

On the other hand, we have specific plugins to perform specific tasks in an external system (e.g. create an argocd application in an ArgoCD instance, pull a Git repository to obtain templates, etc).

NOTE: It is important to take into account that multiple frontend plugins have backend plugins associated in order to be able to display third-party systems information in the Red Hat Developer Hub Portal.

## Create Templates

The Software Templates part of Backstage is a tool that can help clients to create Components inside Backstage. By default, it has the ability to load skeletons of code, template in some variables, and then publish the template to some locations like GitHub or GitLab.

This templates include all information required to create the different components in the software piece and define the required steps to create this resources.

For example, a template can perform the following actions:

* Obtain some templates from a repository
* Render these templates locally (Red Hat Developer Hub)
* Push the final files to a Git repository
* Create an ArgoCD application from a descriptor rendered
* Create a Software Catalog item to save all components information and some additions links

NOTE: Please review the following (file)[./examples/template.yaml] to see an example.

NOTE: In order to be able to display all information about the components generated by a template instantiation, a catalog item will be created at the finish of the process.

## Software Catalog Item

The Backstage Software Catalog is a centralized system that keeps track of ownership and metadata for all the software in your ecosystem (services, websites, libraries, data pipelines, etc).

NOTE: Please review the following (file)[./examples/catalog-info.yaml] to see an example.

## Jump App Application

Jump App is a microservice-based application created to emulate an enterprise application complex architecture with multi environments. This app allows users to configure a set of jumps between components and generate a continuous traffic flow defining the number of retries and their span of time.

In order to emulate a workflow in a company to onboard new Jump App application to test environments, it required to create a new template in Red Hat Developer Hub.

WIP

## Author

Asier Cidon
