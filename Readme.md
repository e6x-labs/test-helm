# Commands for helm

## Ran commands from repo's root.

```
helm create nginx-chart
cd nginx-chart
rm -rf templates/*
```
## Created below files in templates folder
- configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! I got deployed in {{ .Values.env.name }} Environment using Helm Chart </h1>
    </html

```

- deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```
- service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  selector:
    app.kubernetes.io/instance: {{ .Release.Name }}
  type: {{ .Values.service.type }}
  ports:
    - protocol: {{ .Values.service.protocol | default "TCP" }}
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
```

## Ran Validation
### Command
```
helm lint .
```
### Output
```
==> Linting .
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

### Command
```
helm template .
```
### Output

```
---
# Source: nginx-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: release-name-index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! I got deployed in dev Environment using Helm Chart </h1>
    </html
---
# Source: nginx-chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name-service
spec:
  selector:
    app.kubernetes.io/instance: release-name
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9000
---
# Source: nginx-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-chart
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```
### Command

```
helm install --dry-run my-release nginx-chart
```
### Output

```
NAME: my-release
LAST DEPLOYED: Sat May  6 00:03:00 2023
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: nginx-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-release-index-html-configmap
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! I got deployed in dev Environment using Helm Chart </h1>
    </html
---
# Source: nginx-chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-release-service
spec:
  selector:
    app.kubernetes.io/instance: my-release
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9000
---
# Source: nginx-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-release-nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-chart
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```
## Indexing of helm chart

```
helm repo index .  --url https://raw.githubusercontent.com/e6x-labs/test-helm/main/
```

## Adding the helm chart from github repo  
```
helm repo add nginx-chart https://raw.githubusercontent.com/e6x-labs/test-helm/main/
```

## Deploying the helm chart 

### Command
```
helm install my-nginx nginx-chart
```

### Output
```
  NAME: my-nginx
  LAST DEPLOYED: Sat May  6 00:11:34 2023
  NAMESPACE: default
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
```

## Validation of Installation

### Command
```
helm ls -aA | grep nginx
```

### Output
```
my-nginx default  1 	2023-05-06 00:11:34.248627 +0530 IST	deployed	nginx-chart-0.1.0      	1.0.0
```

### Command

```
kubectl get all | grep my-nginx
```
### Output

```
pod/my-nginx-nginx-664dfff499-rpvck             1/1     Running     0              7m27s
pod/my-nginx-nginx-664dfff499-xf8bb             1/1     Running     0              7m27s
service/my-nginx-service                 ClusterIP   10.103.13.170    <none>        80/TCP                         7m28s
deployment.apps/my-nginx-nginx             2/2     2            2           7m28s
replicaset.apps/my-nginx-nginx-664dfff499             2         2         2       7m27s


```
  