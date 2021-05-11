---
title: 'Monitoring'
date: 2019-02-11T19:27:37+10:00
weight: 4
summary: "It is very important to know the status of your resources and your services. You need to collect metrics and logs from your services, as well as your Kubernetes (K3S) server."
---

### Install Prometheus and Grafana

##### Install with Git
First you have to clone the kube prometheus project
```
$> git clone https://github.com/coreos/kube-prometheus.git
```

Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
```
$> cd kube-prometheus
$> kubectl create -f manifests/setup 
$> kubectl create -f manifests/
```
##### Add traefik metrics
Create service metrics to allow prometheus get more services metrics
```
$> kubectl apply -f $git_path/service-metrics/traefik-service-metric.yaml
```

### Install Loki and Promtail

##### Install loki and promtail with helm:
```
helm repo add loki https://grafana.github.io/loki/charts
help repo update
helm fetch --untar loki/loki-stack
cd loki-stack
cp values.yaml my-values.yaml
mkdir manifests
```
The config file is already well configured, we can just deploy it without any changes: 

```
helm template  -f my-values.yaml --output-dir ./manifests --namespace monitoring --name loki .
kubectl apply --recursive -f ./manifests
```

##### Configure loki datasource in Grafana
Go to `https://grafana.etna.student`, next go to configuration -> datasources
and add a new type "loki".
Put as URL: `http://loki.monitoring.svc:3100` and save

### Use Loki
Go into grafana dashboard, in explorer menu, and choose loki as a source.

### See Grafana's dashboard
Go into grafana dashboard. In Dashboard menu, click on manage and choose your dashboard in the CLO4 directory.
