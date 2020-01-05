# Kubernetes @HOVER

## Clusters
We run clusters in EKS and GKE.

> TODO: Add infos for how to access the clusters and how we use them as well as permissions

## Manifests
The manifests for our applications reside in https://github.com/hoverinc/k8s-applications. We utilize [Kustomize](https://kustomize.io) and [ArgoCD](https://argoproj.github.io/argo-cd/) to build and apply manifests to our clusters.

### Style
Follow these guidelines when creating manifests:
* Use `.yaml` as an extension for your manifests
* Use the `---` marker in the beginning of your manifests
* Use two spaces for indentation
* Do not indent sequences

good:
```yaml
apiVersion: extensions/v1beta1
kind: Example
metadata:
  name: my-app-web
spec:
  examples:
  - foo
  - bar
```

bad:
```yaml
apiVersion: extensions/v1beta1
kind: Example
metadata:
    name: my-app-web
spec:
    examples:
        - foo
        - bar
```

### Resources
#### Labels
Apply the following labels to your resources:
* `app`: The name of the application (e.g. `my-app`)
* `env`: The runtime-environment the resources are deployed in (e.g. `staging`, `production`, ....)
* `tier`: The tier for the resource (e.g. `backend`, `frontend`, `rails`, `database`, `redis`, ...)
* `component`: A classifier for the resource within a tier (e.g. `web`, `worker`, ...)

#### Naming
Don't append the `kind` of a resource to its name:
* A `Secret` for the application `my-app` would simply be named `my-app` and not `my-app-secrets`

Use meaningful suffixes for your resources where appropriate:
* Usually this is your `component`, append the tier to the name (e.g. ` my-app-worker`, `my-app-web`)



### Example
The following `Deployment` can be used as a reference for style and naming:
```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-app-web
  labels:
    app: my-app
    tier: backend
    component: web
spec:
  template:
    metadata:
      name: my-app
      labels:
        app: my-app
        tier: backend
        component: web
    spec:
      containers:
      - image: "my-app:latest"
        name: my-app
        imagePullPolicy: Always
        resources:
          requests:
            memory: "1024Mi"
            cpu: "500m"
          limits:
            cpu: "2000m"
            memory: "2048Mi"
        ports:
        - containerPort: 3000
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: my-app
              key: POSTGRES_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-app
              key: POSTGRES_PASSWORD
        envFrom:
        - configMapRef:
            name: my-app
```