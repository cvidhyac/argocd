# GitOps with ArgoCD

https://argo-cd.readthedocs.io/en/stable/getting_started/

## Get Argo CLI

`brew install argocd`

## Local Argo Install

- HA and non-HA manifests available. For local development, we only require non-HA `install.yaml`
  Start the k8s cluster, and run:

The following helm chart install the non-HA version by default:

```shell
helm repo add argo https://argoproj.github.io/argo-helm`
helm install new-argo argo/argo-cd --version 4.8.0
```
Check argo manifests are installed : `k get all -n argocd`

**Tip**

Use Default namespace in the k8s cluster, the local argocli `add cluster` has issues loading
different k8s contexts.


## Enable Access for Argo UI locally

- Change the argocd server to NodePort

```shell
k edit svc new-argo-argocd-server -n argocd
```

- Access the UI : https://127.0.0.1:NODE_PORT
- Login Username : `admin`
- Login Password : Find login password using
  `k get secret -n argocd argocd-initial-admin-secret -o json | jq -r .data.password | base64 -d | pbcopy`

## Argo Terminologies / Concepts

| Terminology             | Description                                                                                                |
|-------------------------|------------------------------------------------------------------------------------------------------------|
| Application             | A CRD that needs to be deployed to a destination k8s cluster                                               |
| Application Source Type | Tools to be used to deploy the app, eg. gitlab, helm, kustomize                                            |
| Project                 | Logical grouping for the applications                                                                      |
| Target State            | Desired state in the K8s cluster - What resources to be created?                                           |
| Live State              | Current state of the K8s resources in the cluster - pods, cm, secrets                                      |
| Sync                    | Process of making an app move from current state to desired state, eg. applying the changes to k8s cluster |
| Sync Status             | Status of the sync                                                                                         |
| Sync Operation status   | Did the sync operation fail or succeed?                                                                    |
| Refresh State           | Compare the latest git code with live state to determine configuration drift, trigger sync if needed       |
| Health                  | ArgoCD provides options to check health of the app, can the app serve traffic ?                            |

## Argo CLI commands


### Create new app

```shell
argocd app create webapp-nginx --repo https://github.com/cvidhyac/learn-helm.git \
 --path ./webapp-nginx --dest-namespace default --dest-server https://kubernetes.default.svc \
 --directory-recurse --validate=false --values-literal-file values.yaml --upsert
 ```

Sync app - `argocd app sync webapp-nginx`
List apps -  `argocd app list`
List projects - `argocd proj list`
Get a project configuration as yaml - `argocd proj get project_name -o yaml`

### Create new project

- Custom projects can be created via yaml, or from UI with ALLOW/DENY rules.
- For an example, see - [webapp-proj.yaml](./webapp-proj.yaml)

### PR title checker

See - https://github.com/marketplace/actions/pr-title-checker
If this plugin is enabled, it can then help Organize your PR into one of the following categories:

- `fix` for bugfixes
- `chore` unit tests / improvements
- `feat` new features and enhancements
- `docs` for doc PR's
