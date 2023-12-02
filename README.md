# GitOps with ArgoCD

https://argo-cd.readthedocs.io/en/stable/getting_started/

## Get Argo CLI

ArgoCLI - `brew install argocd`
Kustomize - `brew install kustomize`

Run `argocd version` and verify it runs without errors

## Local Argo Install

- HA and non-HA manifests available. For local development, we only require non-HA `install.yaml`
  Start the k8s cluster, and run:

```shell
kubectl create ns argocd
kubectl config set-context --current --namespace=argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.5.8/manifests/install.yaml
```

VERSION=v2.4.11
curl -sSL -o
argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

## Enable Access for Argo UI locally

- Change the argocd server to NodePort

```shell
kubectl edit svc argocd-server -n argocd
```

- Access the UI : https://127.0.0.1:NODE_PORT
- Login Username : `admin`
- Login Password : Find login password using
  `k get secret -n argocd argocd-initial-admin-secret -o json | jq -r .data.password | base64 -d | pbcopy`

## Argo CLI login to argo server

`argocd login 127.0.0.1:node_port`

Run the argo CLI command : `argocd app list` and verify it runs without errors

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

| Action                | Command                           |
|-----------------------|-----------------------------------|
| List clusters         | `argocd cluster list              |
| List apps             | `argocd app list`                 |
| Sync app              | `argocd app sync app_name`        |
| Delete app            | `argocd app delete app_name`      |
| Get app configuration | `argocd proj get app_name -oyaml` |

### Create new app

```shell
argocd app create webapp-nginx --repo https://github.com/cvidhyac/learn-helm.git \
 --path ./webapp-nginx --dest-namespace default --dest-server https://kubernetes.default.svc \
 --directory-recurse --validate=false --values-literal-file values.yaml --upsert
 ```

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

## Argo Reconciliation loop

- Argo depends on `Reconciliation loop` to trigger sync live state to desired state.
- Recon loop can be set in 2 ways :
    1. Pull
        - Set `ARGOCD_RECONCILIATION_TIMEOUT` in the `argocd-cm` configmap, and restart the deploy
          for `argocd-repo-server`.
        - The argocd-repo-server would now poll the git repo for the `timeout.reconciliation`
          parameter and trigger sync.
    2. Push
        - Define a Webhook in the git provider (github, gitlab etc.,) configured to the
          argocd-repo-server push events to the ArgoCD api endpoints.
        - Now, the git webhook will execute on the git event and push events to argo server.