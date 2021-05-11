---
title: 'Secure web with Traefik'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 3
summary: "For the security and confidentiality of exchanges, secure the WordPress site by configuring the HTTPS access. Obtaining the certificate should be done automatically via Let’sEncrypt."
---

#### Install Helm
Members of the Helm community have contributed a Helm package for Apt. This package is generally up to date.
```
$> curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
$> sudo apt-get install apt-transport-https --yes
$> echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
$> sudo apt-get update
$> sudo apt-get install helm
```

#### Deploy Traefik
First, you’ll need to add the official Helm repository to your Helm client. You can do that by issuing the following command:
```
$> helm repo add traefik https://helm.traefik.io/traefik
$> helm repo update
```

Then, you’ll need to create a Kubernetes namespace:
```
$> kubectl create namespace traefik
```
Now it's time to deploy Traefik! The following command will install Traefik in the traefik namespace with the custom configuration

``` 
$> helm install traefik traefik/traefik --namespace=traefik --values=helm-values/traefik-values.yaml
```

#### Deploy BasicAuth Middleware for Hugo
First create the secret
```
$> kubectl apply -f secrets/hugo-secret.yaml
```
Then deploy the middleware
```
$> kubectl apply -f middlewares/hugo-basicauth.yaml
```

#### Deploy Ingress Routes
We can finally create all the routes for traefik. Tls is made using Letsencrypt HTTP Challenge
```
$> kubectl apply -f ingress-routes/grafana-ingressroute.yaml
$> kubectl apply -f ingress-routes/metrics-ingressroute.yaml
$> kubectl apply -f ingress-routes/traefik-ingressroute.yaml
$> kubectl apply -f ingress-routes/wordpress-ingressroute.yaml
$> kubectl apply -f ingress-routes/hugo-ingressroute.yaml
$> kubectl apply -f ingress-routes/prometheus-ingressroute.yaml
$> kubectl apply -f ingress-routes/vault-ingressroute.yaml
```

Don't forget to add in your hosts files fqdn entries if you are using local domain:
```
172.16.231.1 web.etna.student
172.16.231.1 traefik.etna.student
172.16.231.1 prometheus.etna.student
172.16.231.1 grafana.etna.student
172.16.231.1 hugo.etna.student
172.16.231.1 vault.etna.student
```

You can monitor your traefik at:
`https://traefik.etna.student` for this exemple
