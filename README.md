# Wordpress-k8s

Wordpress-k8s is a project to deploy Wordpress application on Kubernetes featuring MySQL and Monitoring solutions, such as Loki Promtail Prometheus and Grafana

To use this project you will need to have Kubernetes installed. If you don't already have installed it, please follow the official [installation guide](https://kubernetes.io/docs/setup/)

Also you have to download the project via git:
```
git clone https://rendu-git.etna-alternance.net/module-7830/activity-42973/group-855829 clo4
```

#### NFS

##### NFS Server

First of all, you have to configure the kube master as an NFS Server 
```
$> apt-get install nfs-kernel-server nfs-common
```
Then, you have to share some directories that will be used by the kube nodes, 

Create the directories to be shared:
```
$> mkdir /mnt/wordpress
$> mkdir /mnt/mariadb
$> mkdir /mnt/hugo
```

edit `/etc/exports` file:
```
/mnt/wordpress 172.16.231.14(rw,sync,no_subtree_check,no_root_squash) 172.16.231.26(rw,sync,no_subtree_check,no_root_squash)
/mnt/mariadb 172.16.231.14(rw,sync,no_subtree_check,no_root_squash) 172.16.231.26(rw,sync,no_subtree_check,no_root_squash)
/mnt/hugo 172.16.231.14(rw,sync,no_subtree_check,no_root_squash) 172.16.231.26(rw,sync,no_subtree_check,no_root_squash)
```
(replace by your nodes IPs)

Then confirm your changes:
```
$> exportfs -rv
$> /etc/init.d/nfs-kernel-server start
```

##### NFS Clients
On the kube nodes, mount the shares volumes
```
$> apt-get install nfs-common
```

Create the directories that are going to be used as the destination of the sharing:
```
$> mkdir /mnt/wordpress
$> mkdir /mnt/mariadb
$> mkdir /mnt/hugo
```

Then add this lines to the `/etc/fstab` file
```
172.16.231.1:/mnt/mariadb  /mnt/mariadb   nfs auto,rw,user,soft 0  0
172.16.231.1:/mnt/wordpress  /mnt/wordpress   nfs auto,rw,user,soft 0  0
172.16.231.1:/mnt/hugo  /mnt/hugo   nfs auto,rw,user,soft 0  0
```

Then mount the nfs shares
```
$> mount -a
```

#### Deploy K8S Ressources
On the kube master, create the secret:

```
$> cd clo4/
```

```
$> kubectl apply -f secrets/mariadb-secret.yaml
```

The persistent volumes
```
$> kubectl apply -f volumes/wordpress-volume.yaml
$> kubectl apply -f volumes/mariadb-volume.yaml
$> kubectl apply -f volumes/hugo-volume.yaml
```

Then create the volume claims
```
$> kubectl apply -f volume-claims/wordpress-claim.yaml
$> kubectl apply -f volume-claims/mariadb-claim.yaml
$> kubectl apply -f volume-claims/hugo-claim.yaml
```

Create the deployments
```
$> kubectl apply -f deployments/wordpress-deployment.yaml
$> kubectl apply -f deployments/mariadb-deployment.yaml
$> kubectl apply -f deployments/hugo-deployment.yaml
```
Create the services
```
$> kubectl apply -f services/wordpress-service.yaml
$> kubectl apply -f services/mariadb-service.yaml
$> kubectl apply -f services/hugo-service.yaml
```

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
