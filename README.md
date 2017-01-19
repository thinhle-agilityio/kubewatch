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

To modify what notifications you get, update the `kubewatch` ConfigMap and turn on and off (true/false) resources:

```
resource:
      deployment: false
      replicationcontroller: false
      replicaset: false
      daemonset: false
      services: true
      pod: true
```

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
