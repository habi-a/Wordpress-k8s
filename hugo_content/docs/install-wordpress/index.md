---
title: 'Install Wordpress & Docs'
date: 2019-02-11T19:27:37+10:00
weight: 2
summary: "Deploy a Wordpress service, with its persistent database. The data will need to be retained after a restart of the Kubernetes service or node."
---

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
