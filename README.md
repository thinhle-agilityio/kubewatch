# Kubewatch

`kubewatch` is a Kubernetes watcher that currently publishes notification to Slack. Run it in your k8s cluster, and you will get event notifications in a slack channel. It use slack's webhooks URL instead of API token and channel.

## Run kubewatch in a Kubernetes cluster

In order to run kubewatch in a Kubernetes cluster quickly, the easiest way is for you to create a [ConfigMap](https://github.com/thinhle-agilityio/kubewatch/blob/master/kubewatch-configmap.yaml) to hold kubewatch configuration.

An example is provided at [`kubewatch-configmap.yaml`](https://github.com/thinhle-agilityio/kubewatch/blob/master/kubewatch-configmap.yaml), do not forget to update your own slack webhook url parameters. Alternatively, you could use secrets.

Create k8s configmap:

```console
$ kubectl create -f kubewatch-configmap.yaml
```
Create the [Pod](https://github.com/thinhle-agilityio/kubewatch/blob/master/kubewatch.yaml) directly, or create your own deployment:

```console
$ kubectl create -f kubewatch.yaml
```

A `kubewatch` container will be created along with `kubectl` sidecar container in order to reach the API server.

Once the Pod is running, you will start seeing Kubernetes events in your configured Slack channel. Here is a screenshot:

![slack](./docs/slack.png)

To modify what notifications you get, update the `kubewatch-configmap.yaml` ConfigMap and turn on and off (true/false) resources:

```
resource:
      deployment: false
      replicationcontroller: false
      replicaset: false
      daemonset: false
      services: true
      pod: true
```

- Kubewatch secret `KW_SLACK_WEBHOOK` will be used to config webhook url to a specific channel
- If kubewatch-configmap have `handler/slack/webhook`, it will override webhook url in secret


## Building

### Building with go

- you need go v1.5 or later.
- if your working copy is not in your `GOPATH`, you need to set it accordingly.

```console
$ go build -o kubewatch main.go
```

You can also use the Makefile directly:

```console
$ make build
```

### Building with Docker

Buiding builder image:

```console
$ make builder-image
```

Using the `kubewatch-builder` image to build `kubewatch` binary:

```console
$ make binary-image
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
kubewatch           latest              f1ade726c6e2        31 seconds ago       33.08 MB
kubewatch-builder   latest              6b2d325a3b88        About a minute ago   514.2 MB
```

## Download kubewatch package

```console
$ go get -u github.com/thinhle-agilityio/kubewatch
```

## Configuration
Kubewatch supports `config` command for configuration. Config file will be saved at $HOME/.kubewatch.yaml

### Configure slack

```console
$ kubewatch config slack --webhook <webhook_url>
```

### Configure resources to be watched

```console
// rc, po and svc will be watched
$ kubewatch config resource --rc --po --svc

// only svc will be watched
$ kubewatch config resource --svc
```

### Environment variables
You have an altenative choice to set your SLACK webhook url via environment variables:

```console
$ export KW_SLACK_WEBHOOK='XXXXXXXXXXXXXXXX'
```

### Run kubewatch locally

```console
$ kubewatch
```

### Update for Kubernetes >= 1.6

From Kubernetes from 1.6 and upper, you have to grant permission to pod for accessing the kube API server through [RBAC Authorization](https://kubernetes.io/docs/admin/authorization/rbac/). Pod use Service Account (usually default Service Account) to access the API server. But Kubernetes grant no permissions to service account outside the `kube-system` namespace.

So you have to create below ClusterRole and ClusterRoleBinding (find it under ./kubewatch-rbac.yaml):

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
    name: kubewatch
    namespace: default
rules:
- apiGroups:
  - ""
  - "extensions"
  resources:
  - configmaps
  - secrets
  - services
  - endpoints
  - ingresses
  - nodes
  - pods
  verbs:
  - list
  - get
  - watch
- apiGroups:
  - "*"
  - ""
  resources:
  - events
  - certificates
  - secrets
  verbs:
  - create
  - list
  - update
  - get
  - patch
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kubewatch
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubewatch
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
```
