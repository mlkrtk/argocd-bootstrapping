# ArgoCD - managing cluster add-ons with ApplicationSet controller 

This is a ArgoCD `cluster add-ons` repository.  `ApplicationSet controller` allows DevOps team the ability to automatically create a large, diverse set of Argo CD Applications, across a significant number of clusters, and manage those Applications as a single unit.

In the cluster add-on use case, DevOps team is responsible for provisioning cluster add-ons to one or more Kubernete clusters: `cluster addons` are operators such as the Extrernal Secrets Operator (ESO) or controllers such as the AWS Load Balancer Controller.


## Instructions 

Step #1 Update credentials in ~/.kube/config, if needed.

```
aws eks --region <AWS_REGION> update-kubeconfig --name <ARGOCD_SERVER_EKS_CLUSTER_NAME>` 
```

Step #2 Login to Argo CD via CLI.

```
argocd login <ip>:<port>
```

Step #3 Create intial root addons project and recursive application

Clone git repo https://github.com/mlkrtk/argocd-bootstrapping.git, create initial addons root project and recursive application.

```
git clone https://github.com/mlkrtk/argocd-bootstrapping.git

( cd argocd-infra-bootstrapping )
kubectl apply -f 'addon-base-root/addon-base-root/*.yaml'
```  

```
argocd proj list

NAME                          DESCRIPTION                   DESTINATIONS                           SOURCES                                                                               CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES
addon-base-root               Addon base root               https://kubernetes.default.svc,argocd  https://github.com/mlkrtk/argocd-bootstrapping.git  */*                         <none>                        <none>          disabled
```

```
argocd app list

NAME                                          CLUSTER                         NAMESPACE                     PROJECT                       STATUS   HEALTH   SYNCPOLICY  CONDITIONS       REPO                                                                                  PATH                                                     TARGET
addon-base-root                               https://kubernetes.default.svc  argocd                        addon-base-root               Synced   Healthy  Auto-Prune  <none>           https://github.com/mlkrtk/argocd-bootstrapping.git  addon-base                                               dev
```


#Step 4 Add credentials template URL for BitBucket

```
argocd repocreds add https://github.com/mlkrtk/ --username mlkrtk --password <TOKEN>
```

#Step 5 Join a new cluster to ArgoCD server

Update kube config to use target app server e.g. new dev server and join to Argo CD with CLI. 
```
aws eks --region <AWS_REGION> update-kubeconfig --name <APP_SERVER_EKS_CLUSTER_NAME>`

argocd cluster add arn:aws:eks:ap-south-1:891543987898:cluster/opq-dev --name opq-dev --label project/opq=true --label region=ap-south-1 --label provider=aws --label cluster-addons=true --label env=dev

WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `arn:aws:eks:ap-south-1:891543987898:cluster/opq-dev` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0002] ServiceAccount "argocd-manager" already exists in namespace "kube-system"
INFO[0002] ClusterRole "argocd-manager-role" updated
INFO[0002] ClusterRoleBinding "argocd-manager-role-binding" updated
Cluster 'https://8344F32EA7C641C9157D6CAE0ADF087D.yl4.ap-south-1.eks.amazonaws.com' added

argocd cluster list
SERVER                                                                       NAME             VERSION  STATUS      MESSAGE                                                  PROJECT
https://8344F32EA7C641C9157D6CAE0ADF087D.yl4.ap-south-1.eks.amazonaws.com  opq-dev           Unknown     Cluster has no applications and is not being monitored.
https://kubernetes.default.svc                                               in-cluster       1.22+    Successful
```

#Step 6 Login to ArgoCD WebUI and look at the applications deployed to target cluster

## Folder Structure

* `addon-base-root/addon-base-root` defines top level project and application used for addons. Only deployed once to ArgoCD parent server.
* `addon-base` defines applicationset and project for each application. 
* `addon-apps` define configuration parameters for each applicationset.


```
|-- README.md
|-- addon-apps
|   |-- argo-rollouts
|   |   `-- applicationsets
|   |       `-- appset.yaml
|   |-- aws-load-balancer-controller
|   |   |-- applicationsets
|   |   |   `-- appset.yaml
|   |   `-- cluster-config
|   |       |-- opq-dev
|   |       |   `-- config.yaml
|   |       `-- opq-qa
|   |           `-- config.yaml
|   |-- cluster-autoscaler
|   |   |-- applicationsets
|   |   |   `-- appset.yaml
|   |   `-- cluster-config
|   |       |-- opq-dev
|   |       |   `-- config.yaml
|   |       `-- opq-qa
|   |           `-- config.yaml
|   `-- external-secrets
|       |-- applicationsets
|       |   `-- appset.yaml
|       `-- cluster-config
|           |-- opq-dev
|           |   `-- config.yaml
|           `-- opq-qa
|               `-- config.yaml
|-- addon-base
|   |-- argo-rollouts
|   |   |-- appset-app.yaml
|   |   `-- project.yaml
|   |-- aws-load-balancer-controller
|   |   |-- appset-app.yaml
|   |   `-- project.yaml
|   |-- cluster-autoscaler
|   |   |-- appset-app.yaml
|   |   `-- project.yaml
|   `-- external-secrets
|       |-- appset-app.yaml
|       `-- project.yaml
|-- addon-base-root
|   |-- app.yaml
|   `-- project.yaml
|-- opq-apps
|   `-- tetris
|       |-- applicationsets
|       |   `-- appset.yaml
|       `-- cluster-config
|           |-- opq-dev
|           |   `-- config.yaml
|           `-- opq-qa
|               `-- config.yaml
|-- opq-base
|   `-- tetris
|       |-- appset-app.yaml
|       `-- project.yaml
`-- opq-base-root
    |-- app.yaml
    `-- project.yaml
```
