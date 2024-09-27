# Introduction to Kubernetes Helm

## What is Helm?

Helm is a **package manager** for Kubernetes, designed to streamline the deployment and management of applications on a Kubernetes cluster. It uses a concept called **Helm charts**, which are pre-configured packages that include the necessary Kubernetes resources (like Deployments, Services, and ConfigMaps) needed to run an application.

By providing reusable templates and managing versioning, Helm simplifies the process of deploying and updating applications, handling rollbacks, and managing application configuration, all in a Kubernetes environment.

## Key Features of Helm

- **Chart-based Packaging**: Helm charts package Kubernetes resources, allowing you to easily distribute, share, and reuse configurations.
- **Versioning and Rollbacks**: Helm supports release versioning and rollback, ensuring you can revert to a previous configuration if an upgrade fails.
- **Simplified Upgrades**: Helm makes it easier to upgrade your application with just a few commands, ensuring minimal downtime.
- **Dependency Management**: Helm manages dependencies between different charts, helping in multi-tiered applications.
- **Templating**: Helm allows you to dynamically generate Kubernetes manifests using templates, making it flexible to support different environments (dev, staging, production).

## Use Cases for Helm

1. **Application Deployment**  
   Helm is used to simplify the deployment of complex applications by providing a package (chart) that includes all necessary resources.
   
   Example:  
   Deploying a multi-tier application with a database, web server, and caching layer becomes easier using Helm charts instead of managing individual Kubernetes manifests.

2. **CI/CD Pipelines**  
   Helm integrates well with CI/CD pipelines, automating application updates, managing different environments, and making application rollbacks quick and easy.

   Example:  
   A continuous integration pipeline using Helm can deploy a new version of an app to the dev environment for testing, then promote it to staging, and finally to production.

3. **Environment Configuration Management**  
   Helm can manage different configurations for different environments (e.g., dev, staging, prod) using its templating system and `values.yaml` files.

   Example:  
   You can define different configurations (like database URLs, replica counts, or API keys) and apply them to different environments using Helm.

4. **Application Rollback**  
   Helm supports versioned releases, allowing you to roll back to a previous version of your application if an upgrade goes wrong.

   Example:  
   If a deployment to production has issues, you can use Helm to quickly roll back to the previous working version.

## Helm Architecture

Helm consists of two main components:
1. **Helm CLI**: The command-line interface where you interact with Helm to install, upgrade, and manage releases.
2. **Helm Charts**: Pre-packaged application configurations that can be customized with values and templates.

When you use Helm to install or upgrade an application, it creates a **Helm release**. A release is a specific instance of a Helm chart, tied to a particular version and configuration.

## Basic Helm Commands

Here are some key commands to help you get started with Helm:


# Install ArgoCD in k8s
https://argo-cd.readthedocs.io/en/stable/getting_started/

# Install Helm:

  ```bash
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  ```

**Step-01a: Install Argo CD on the cluster using Helm as follows:**

  ```bash
  helm repo add argo https://argoproj.github.io/argo-helm
  kubectl create namespace argocd
  helm install argocd -n argocd argo/argo-cd
  ```

**Step-01b: Create argocd namespace and install argocd using kubectl**
```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
**Step-02: Access ArgoCD UI**
```shell
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd --address="0.0.0.0"
```
**Step-03: Login with admin user and below token (as in documentation):**
```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
or 
kubectl -n argocd get secret argocd-initial-admin-secret -oyaml        #copy the base64 encoded password
echo <password> | base64 --decode    #ignore the % at the end
```

**Step-04: Install ArgoCD CLI**

```bash
  wget https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-linux-amd64
  sudo mv argocd /usr/local/bin/argocd
  chmod +x /usr/local/bin/argocd
```
**Step-05: Login to argocd using the following command.**

  ```bash
  argocd login localhost:8080 --insecure
  argocd login <node-ip>:8080 --insecure
  ```

**Step-06: Verify that you are logged in by running:**

  ```bash
  argocd repo list
  ```

**Step-07: Change the password using the following command:**

  ```bash
  argocd account update-password \
  --current-password # current password \
  --new-password # new password
  ```
**Step-08: Add a repo secret to argocd**
a) create a personal access token in github /gitlab
b) Copy the token
c) On the terminal, add a secret to argocd by running 
```shell
argocd repo add "<YOUR-REPO>" --username "<USERNAME>" --password "<PAT>"
argocd repo add "https://github.com/Wemadevops/argo-testing.git" --username "wemadevops" --password "ghp_fpmWk5cVHmnmuB0Zn5uzxZ2Qv3ms"
```

or
```yaml
#secret.yaml

apiVersion: v1
kind: Secret
metadata:
   name: argocd-repo
   namespace: argocd
   labels:
      argocd.argoproj.io/secret-type: repository
stringData:
   type: git
   url: https://github.com/yourrepo.git
   password: <your-token>
   username: <your-username>
 ```

**Step-09: Deploy the application to the cluster**
Create and deploy an application in the cluster to `argodc` namespace
- Modify the `application.yaml` with your git repository and apply to the cluster.
```
kubectl apply -f application.yaml
```
```yaml
#application.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default
  source:
     repoURL: https://github.com/yourrepo.git
     targetRevision: HEAD
     path: prod
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

**Step-10: Push your manifest files to your repo**

Create a Git workflow as follows:
```bash
git checkout -b feature/app
# modify the deployment replicas to be three
git add prod/deployment.yaml
git commit -m "Increases the replica count"
git push --set-upstream origin feature/app
```

```yaml
#prod/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
     app: myapp
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp
    spec:
       containers:
       - name: myapp
         image: wemadevops/kube-frontend-nginx:1.0.0
         ports:
         - containerPort: 8080
```
```yaml
#service.yaml

apiVersion: v1
kind: Service
metadata:
   name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```
**Step-11: Create a PR and after its merged, sync your application**

Go back to the terminal and run the following:
```bash
argocd app sync nginx
kubectl get pods
```
or go onto the UI and synchronize the application