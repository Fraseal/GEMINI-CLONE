# Jenkins with Helm on Kubernetes — Installation & Helm Commands

> **GitHub-friendly guide:** All executable commands are shown in fenced `bash` code blocks so they are easy to spot, copy, and paste.

This guide covers installing Jenkins on Kubernetes using the official Jenkins Helm chart, plus the most useful Helm commands for day-to-day work.

> The official Jenkins documentation uses Helm 3 for Kubernetes installations, and the Jenkins chart repository is `https://charts.jenkins.io`. citeturn0search0turn0search1

---

# 1. Prerequisites

Make sure Kubernetes and Helm are installed:

```bash
kubectl version --client
helm version
```

Check that your cluster is reachable:

```bash
kubectl get nodes
```

If you are using Minikube:

```bash
minikube status
```

If you are using Kind:

```bash
kind get clusters
kubectl cluster-info
```

---

# 2. Add the Jenkins Helm Repository

Add the official Jenkins chart repository:

```bash
helm repo add jenkins https://charts.jenkins.io
```

Update the repository:

```bash
helm repo update
```

Check the available Jenkins chart:

```bash
helm search repo jenkins
```

Expected output will contain something similar to:

```text
NAME             CHART VERSION    APP VERSION
jenkins/jenkins  <version>        <version>
```

---

# 3. Create a Jenkins Namespace

Create a dedicated namespace:

```bash
kubectl create namespace jenkins
```

Check it:

```bash
kubectl get namespaces
```

---

# 4. Check Jenkins Chart Values

Before installing, you can inspect the default chart values:

```bash
helm show values jenkins/jenkins
```

Save the values to a file:

```bash
helm show values jenkins/jenkins > jenkins-values.yaml
```

You can then edit the file:

```bash
vi jenkins-values.yaml
```

or, if `nano` is installed:

```bash
nano jenkins-values.yaml
```

---

# 5. Simple Jenkins Installation

For a basic installation:

```bash
helm install jenkins jenkins/jenkins -n jenkins
```

Check the Helm release:

```bash
helm list -n jenkins
```

Check Jenkins resources:

```bash
kubectl get all -n jenkins
```

Check pods:

```bash
kubectl get pods -n jenkins
```

Wait until the Jenkins pod becomes `Running` and `Ready`.

---

# 6. Install Jenkins Using a Values File

Create a small custom values file:

```yaml
controller:
  serviceType: NodePort

persistence:
  enabled: true
```

Save it as:

```text
jenkins-values.yaml
```

Install Jenkins:

```bash
helm install jenkins jenkins/jenkins \
  -n jenkins \
  -f jenkins-values.yaml
```

Check:

```bash
helm status jenkins -n jenkins
kubectl get pods -n jenkins
kubectl get svc -n jenkins
```

> For production, configure persistent storage through a suitable Kubernetes StorageClass/PVC rather than relying on temporary pod storage.

---

# 7. Get the Jenkins Admin Password

The exact secret/key can depend on the chart configuration. A common command for the official chart is:

```bash
kubectl exec -n jenkins svc/jenkins -c jenkins -- \
  /bin/cat /run/secrets/additional/chart-admin-password
```

If that path is not available, inspect the secrets:

```bash
kubectl get secrets -n jenkins
```

Then inspect the Jenkins-related secret:

```bash
kubectl describe secret <SECRET_NAME> -n jenkins
```

You can also check the chart's installation notes:

```bash
helm status jenkins -n jenkins
```

---

# 8. Access Jenkins

Check the Jenkins service:

```bash
kubectl get svc -n jenkins
```

For Minikube, you can use:

```bash
minikube service jenkins -n jenkins
```

To find the NodePort:

```bash
kubectl get svc jenkins -n jenkins
```

Example:

```text
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
jenkins   NodePort   10.x.x.x        <none>        8080:3xxxx/TCP
```

Then access:

```text
http://<NODE-IP>:<NODE-PORT>
```

For a quick local tunnel, you can also use:

```bash
kubectl port-forward -n jenkins svc/jenkins 8080:8080
```

Then open:

```text
http://localhost:8080
```

---

# 9. Helm Release Commands

## List releases

All namespaces:

```bash
helm list -A
```

Specific namespace:

```bash
helm list -n jenkins
```

---

## Check release status

```bash
helm status jenkins -n jenkins
```

---

## Get release values

```bash
helm get values jenkins -n jenkins
```

Get all computed values:

```bash
helm get values jenkins -n jenkins -a
```

---

## Get release manifest

```bash
helm get manifest jenkins -n jenkins
```

---

## Get release notes

```bash
helm get notes jenkins -n jenkins
```

---

## Get all release information

```bash
helm get all jenkins -n jenkins
```

---

# 10. Helm Search Commands

Search repositories:

```bash
helm search repo jenkins
```

Search all available charts:

```bash
helm search repo
```

Search Artifact Hub:

```bash
helm search hub jenkins
```

---

# 11. Helm Repository Commands

Add a repository:

```bash
helm repo add <repo-name> <repo-url>
```

Example:

```bash
helm repo add jenkins https://charts.jenkins.io
```

List repositories:

```bash
helm repo list
```

Update repositories:

```bash
helm repo update
```

Remove a repository:

```bash
helm repo remove <repo-name>
```

Example:

```bash
helm repo remove jenkins
```

---

# 12. Helm Install Commands

Basic syntax:

```bash
helm install <release-name> <chart>
```

Example:

```bash
helm install jenkins jenkins/jenkins
```

Install into a namespace:

```bash
helm install jenkins jenkins/jenkins -n jenkins
```

Create the namespace automatically:

```bash
helm install jenkins jenkins/jenkins \
  --namespace jenkins \
  --create-namespace
```

Install using a values file:

```bash
helm install jenkins jenkins/jenkins \
  -n jenkins \
  -f jenkins-values.yaml
```

Set individual values from the command line:

```bash
helm install jenkins jenkins/jenkins \
  -n jenkins \
  --set controller.serviceType=NodePort
```

---

# 13. Helm Upgrade

Upgrade an existing release:

```bash
helm upgrade jenkins jenkins/jenkins -n jenkins
```

Upgrade with a values file:

```bash
helm upgrade jenkins jenkins/jenkins \
  -n jenkins \
  -f jenkins-values.yaml
```

Upgrade and install if the release does not exist:

```bash
helm upgrade --install jenkins jenkins/jenkins \
  -n jenkins \
  --create-namespace
```

This is a very useful command for CI/CD and automation.

---

# 14. Helm Rollback

View release history:

```bash
helm history jenkins -n jenkins
```

Example:

```text
REVISION    STATUS
1           superseded
2           deployed
```

Rollback to revision 1:

```bash
helm rollback jenkins 1 -n jenkins
```

Check the result:

```bash
helm status jenkins -n jenkins
```

---

# 15. Helm Uninstall

Remove the Jenkins Helm release:

```bash
helm uninstall jenkins -n jenkins
```

Check:

```bash
helm list -n jenkins
```

> Important: uninstalling a Helm release does not necessarily mean every persistent data object will be removed. Check PVCs before deleting storage.

Check PVCs:

```bash
kubectl get pvc -n jenkins
```

---

# 16. Helm Template

Render Kubernetes YAML locally without installing:

```bash
helm template jenkins jenkins/jenkins
```

With a namespace:

```bash
helm template jenkins jenkins/jenkins \
  -n jenkins
```

With custom values:

```bash
helm template jenkins jenkins/jenkins \
  -n jenkins \
  -f jenkins-values.yaml
```

This is useful for troubleshooting templates before applying them.

---

# 17. Helm Lint

If you create your own Helm chart:

```bash
helm lint ./my-chart
```

Example:

```bash
helm lint .
```

---

# 18. Download / Pull a Chart

Pull a chart locally:

```bash
helm pull jenkins/jenkins
```

Pull and extract it:

```bash
helm pull jenkins/jenkins --untar
```

This creates a local chart directory that you can inspect.

---

# 19. Package a Helm Chart

Package your local chart:

```bash
helm package ./my-chart
```

Example output:

```text
Successfully packaged chart and saved it to: my-chart-1.0.0.tgz
```

---

# 20. Create a New Helm Chart

Create a starter chart:

```bash
helm create mychart
```

Directory structure:

```text
mychart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
└── .helmignore
```

Install it:

```bash
helm install myrelease ./mychart
```

---

# 21. Helm Dependency Commands

Update chart dependencies:

```bash
helm dependency update ./mychart
```

Build dependencies:

```bash
helm dependency build ./mychart
```

List dependencies:

```bash
helm dependency list ./mychart
```

---

# 22. Debug a Helm Deployment

Use dry-run:

```bash
helm install jenkins jenkins/jenkins \
  -n jenkins \
  --dry-run
```

Use debug output:

```bash
helm install jenkins jenkins/jenkins \
  -n jenkins \
  --dry-run \
  --debug
```

For an existing release:

```bash
helm upgrade jenkins jenkins/jenkins \
  -n jenkins \
  --dry-run \
  --debug
```

---

# 23. Useful Kubernetes Commands with Helm

Check all Jenkins resources:

```bash
kubectl get all -n jenkins
```

Check pods:

```bash
kubectl get pods -n jenkins -o wide
```

Watch pods:

```bash
kubectl get pods -n jenkins -w
```

Check services:

```bash
kubectl get svc -n jenkins
```

Check PVCs:

```bash
kubectl get pvc -n jenkins
```

Check events:

```bash
kubectl get events -n jenkins --sort-by=.lastTimestamp
```

Describe Jenkins pod:

```bash
kubectl describe pod <POD_NAME> -n jenkins
```

View Jenkins logs:

```bash
kubectl logs <POD_NAME> -n jenkins
```

Follow logs:

```bash
kubectl logs -f <POD_NAME> -n jenkins
```

---

# 24. Common Jenkins Helm Troubleshooting

## Pod is Pending

Check:

```bash
kubectl describe pod <POD_NAME> -n jenkins
```

Check PVC:

```bash
kubectl get pvc -n jenkins
```

Check StorageClass:

```bash
kubectl get storageclass
```

---

## Pod is CrashLoopBackOff

Check logs:

```bash
kubectl logs <POD_NAME> -n jenkins
```

Previous container logs:

```bash
kubectl logs <POD_NAME> -n jenkins --previous
```

Describe the pod:

```bash
kubectl describe pod <POD_NAME> -n jenkins
```

---

## Helm release already exists

Error:

```text
cannot re-use a name that is still in use
```

Check:

```bash
helm list -A
```

Either upgrade:

```bash
helm upgrade jenkins jenkins/jenkins -n jenkins
```

or uninstall first:

```bash
helm uninstall jenkins -n jenkins
```

---

# 25. Recommended Jenkins Installation Command

For a simple lab environment:

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update

kubectl create namespace jenkins

helm upgrade --install jenkins jenkins/jenkins \
  --namespace jenkins \
  --create-namespace
```

Then:

```bash
kubectl get pods -n jenkins
kubectl get svc -n jenkins
helm status jenkins -n jenkins
```

---

# 26. Most Important Helm Commands — Quick Cheat Sheet

```bash
# Repository
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm repo list

# Search
helm search repo jenkins
helm show values jenkins/jenkins

# Install
helm install jenkins jenkins/jenkins -n jenkins --create-namespace

# Upgrade
helm upgrade jenkins jenkins/jenkins -n jenkins

# Upgrade or install
helm upgrade --install jenkins jenkins/jenkins -n jenkins --create-namespace

# List
helm list -A

# Status
helm status jenkins -n jenkins

# Values
helm get values jenkins -n jenkins

# Manifest
helm get manifest jenkins -n jenkins

# History
helm history jenkins -n jenkins

# Rollback
helm rollback jenkins 1 -n jenkins

# Template
helm template jenkins jenkins/jenkins -n jenkins

# Uninstall
helm uninstall jenkins -n jenkins

# Chart information
helm show chart jenkins/jenkins
helm show values jenkins/jenkins
helm show readme jenkins/jenkins
```

---

## Official References

Jenkins Kubernetes installation documentation:
https://www.jenkins.io/doc/book/installing/kubernetes/

Official Jenkins Helm charts:
https://github.com/jenkinsci/helm-charts

Jenkins Helm chart repository:
https://charts.jenkins.io/

