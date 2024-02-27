# Openshift Developer Hub KickStart

The idea of this repository is to provide the required information to onboard quickly in Red Hat Developer Hub understanding the different pieces included in this solution.

There is included a use case *Onboard Jump App Application* at the end of this document in order to understand all pieces working together and the configuration workflows.

## Prerequisites

* Openshift 4.14+
* OC Client 4.14+
* Helm Client v3.14.2+

## Developer Hub Setup

In order to install Red Hat Developer Hub, it is required to follow the next steps:

```$bash
## Install Openshift GitOps
$ oc apply -f files/argocd.yaml

## Create the respective Namespaces
$ oc new-project backstage
$ oc new-project jump-app
$ oc label namespace jump-app argocd.argoproj.io/managed-by=openshift-gitops --overwrite

## Create ArgoCD Credentials
$ cp files/secret-template.yaml /tmp/secret.yaml
$ echo "\n  username: $(echo admin | base64)" >> /tmp/secret.yaml
$ echo "  password: $(oc get secret openshift-gitops-cluster -o jsonpath='{.data.admin\.password}' -n openshift-gitops | base64 -d)" >> /tmp/secret.yaml
$ echo "  url: $(oc get route openshift-gitops-server -n openshift-gitops -o jsonpath='{.spec.host}' | base64)" >> /tmp/secret.yaml
$ oc create -f /tmp/secret.yaml

## Install Developer Hub
$ helm repo add openshift-helm-charts https://charts.openshift.io/
$ vi files/values.yaml (*Modify global.clusterRouterBase)
$ helm upgrade -i developer-hub -f files/values.yaml openshift-helm-charts/redhat-developer-hub
```

NOTE: Extract original values.yaml by using command "$ helm show values openshift-helm-charts/redhat-developer-hub"

Please visit the following [link](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#proc-install-rhdh-helm_admin-rhdh) for more information about the installation process via Web Console.

## Install Dynamic Plugins by Red Hat (*Installed by the Developer Hub Setup procedure*)

**This section try to explain how the procedure to install dynamic plugins in Red Hat Developer Hub is but they are installed by the Developer Hub Setup procedure.**

Red Hat Developer Hub, based on Backstage, is a single-page application composed of a set of plugins.

Red Hat has developed some plugins in order to support and allow multiple integration with third party solutions, for example Red Hat GitOps based on Argocd or Red Hat Pipelines based on Tekton.

There are two main groups of dynamic plugins:

* Frontend plugins -> Frontend components that will be used to display specific information in the client's browser.
* Backend plugins -> The backend uses plugins to construct a working backend that the frontend (app) can use to interact with third party systems and/or make specific actions.

Please review the following [link](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#rhdh-supported-plugins) for more information about dynamic plugins supported by Red Hat.

### Find your plugin

First of all, it is important to find information about the plugin once you have decided to use it. For this mission, you can look for information to [Backstage Plugins and Documentation](https://backstage.io/plugins/) or [NPM Official Web][https://www.npmjs.com/] using the plugin name (E.g. @roadiehq/scaffolder-backend-argocd).

Once you are in the NPM documentation, it is possible to find the GitHub repository where it is easy to find more information about the plugin.

NOTE: It is important to identify if the respective plugin requires specific env vars and the specific configuration that will be required in the following section

### Configure the Dynamic Plugins

Once the dynamic plugins have been found, it is time to install and configure them following the next steps:

* Add sensitive information via secrets (E.g. ArgoCD information for plugins)

```$bash
oc create -f /tmp/secret.yaml
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
          - name: NODE_TLS_REJECT_UNAUTHORIZED
            value: "0"

```

### Use Dynamic Plugins

Once the Red Hat Developer Hub pod has been rolled out because of the new configuration inclusion, it is time to use the respective plugin.

It is important to understand the way of interacting with the plugin. For example, it is a plugin that displays specific information in the Red Hat Developer Hub Portal, it maybe does not required extra steps to start playing with it.

On the other hand, we have specific plugins to perform specific tasks in an external system (e.g. create an argocd application in an ArgoCD instance, pull a Git repository to obtain templates, etc).

NOTE: It is important to take into account that multiple frontend plugins have backend plugins associated in order to be able to display third-party systems information in the Red Hat Developer Hub Portal.

## Create Templates (*Installed by the Developer Hub Setup procedure*)

**This section try to explain how the procedure to create templates in Red Hat Developer Hub is but they are installed by the Developer Hub Setup procedure.**

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

The integration between Red Hat Developer Hub and Jump App allows developer to see how the applications have been deployed in the Openshift Clusters via ArgoCD including the following features:

* Display Jump App current state via ArgoCD integration
* Display information about the GitOps repository (URL)
* Display information about the Jump App Golang Backend API

In order to emulate a workflow in a company to onboard new Jump App application to test environments, it required to create a new template in Red Hat Developer Hub.

### Enable Required Plugins (*Installed by the Developer Hub Setup procedure*)

Please review the section "Install Dynamic Plugins" in order to understand the configuration process. 

NOTE: It is important to bear in mind that this procedure modify parameter in the Helm Chart values file and requires a Red Hat Developer Hub pod rollout.

### Create Jump App Application Templates

The Software Templates part of Backstage is a tool that can help you create Components inside Backstage. By default, it has the ability to load skeletons of code, template in some variables, and then publish the template to some locations like GitHub or GitLab.

Templates are stored in the Software Catalog under a kind Template. You can create your own templates with a small yaml definition which describes the template and its metadata, along with some input variables that your template will need, and then a list of actions which are then executed by the scaffolding service.

The most important parts of a template are included in the following list:

* type
* owner
* parameters
* steps

```$yaml
...
spec:
  type: website
  owner: team-a
  parameters:
    - name: Enter some stuff
      description: Enter some stuff
  ...
  steps:
    action: publish:github
    ...
    input:
        ...
        gitAuthorName: ${{ user.entity.metadata.name }}
        gitAuthorEmail: ${{ user.entity.spec.profile.email }}
```

### Enable Templates (*Installed by the Developer Hub Setup procedure*)

**This section try to explain how the procedure to install a new template in Red Hat Developer Hub is but it is installed by the Developer Hub Setup procedure.**

```$bash
    appConfig:
      catalog:
        rules:
          - allow: [Component, System, API, Resource, Location, Template]
        locations:
          - target: https://github.com/acidonper/ocp-rh-developer-hub-setup/blob/master/software-templates/templates.yaml
            type: url
            rules:
              - allow: [Template]
```

## Interesting Links

* [Red Hat Developer Hub Official Documentation](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#snip-customer-support-info_admin-rhdh)
* [Backstage Documentation](https://backstage.io/docs/overview/what-is-backstage)
* [Backstage Plugins and Documentation](https://backstage.io/plugins/)
* [Red Hat Developer Hub - Supported Plugins](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html-single/administration_guide_for_red_hat_developer_hub/index#proc-install-rhdh-helm_admin-rhdh)
* [NPM Packages repository](https://www.npmjs.com/)

## Author

Asier Cidon
