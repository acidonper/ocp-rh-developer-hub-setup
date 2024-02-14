# Jump App Application Template

This repository includes all resources required to deploy Jump App in Openshift using a GitOps model.

## Variables

The following variables are required in the different templates:

- values.component_id -> Red Hat Developer Hub Component ID (*Name*)
- values.ocp_cluster -> OCP Cluster where the app will be deployed
- values.ocp_namespace -> OCP Namespace where the app will be deployed
- values.ocp_apps_domain -> Openshift Domain for applications
- values.gitops_repo -> Repository Url where GitOps resources are hosted
- values.gitops_repo_files -> Repository Url where GitOps resources are hosted to get specific files
- values.gitops_repo_branch -> Repository Branch where GitOps resources are hosted

## Folders

- scaffolders/**/manifests/argocd/ -> Hosts ArgoCD Application that will be rendered per every app
- scaffolders/**/manifests/catalog/ -> Hosts Red Hat Developer Hub Catalog that will be rendered per every app

## Author

Asier Cidon @RedHat
