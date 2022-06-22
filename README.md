This is cluster setup minimal instructions with `microk8s` and attached NFS storage.
First of all install [MicroK8s](https://microk8s.io/docs) on all of the computers in the cluster.

This repository contains:
* Elasticsearch
* Kibana
* Elastic APM
* Fake-s3 (s3 mock server)
* RabbitMQ
* Redis
* Private Registry (docker-registry)

# Master
1. Type `microk8s add-node`
2. Copy the value of the approprite network ip
3. Open new tab in terminal
4. Connect by SSH to the new node
5. Paste output from *stage 1* and paste
   (make sure you copy the command related to the same network both computers attched to)
6. Wait for connection to be established

# New-Node
1. Set static-ip to the new node
2. For enable the cluster to connect to nfs please `sudo apt-get install -y nfs-common`

# In all computers
1. Run this `microk8s kubectl get nodes -o wide | awk '{ print $6 " " $1 }' | tail -n +2 | grep -v $(hostname)`
2. Append the `/etc/hosts` file

# NFS-Server
* Install nfs-kernel-server by `sudo apt install -y nfs-kernel-server`
* Mount new disk/s and create a folder inside of them
* Change owner to nobody by `sudo chown -R nobody:nogroup <path_to_dir>`
* Change the permissions for the folder by `sudo chmod -R 777 <path_to_dir>`
* Add networking settings in `/etc/exports` <br/>
  *(example: `<path_to_dir> <static_ip_of_computer>/24(rw,sync,no_wdelay,root_squash,insecure,no_subtree_check,fsid=0)`)*

More info in [install nfs server on ubuntu](https://www.tecmint.com/install-nfs-server-on-ubuntu/)

# Before Starting to work
## All of these are optional
Add dashboard with the command: `microk8s enable dashboard`
Add RBAC (role cluster management) with: `microk8s enable rbac`
Add dns (dns cluster management) with: `microk8s enable dns`

Before applying new deployment, don't forget to enable cluster-dns by `microk8s enable dns`
It's very recomended to set `alias` of `microk8s kubectl` with just `kubectl` to save time. You can do this with `alias kubectl="microk8s kubectl"`

Run on one of the computers the command `microk8s dashboard-proxy` and follow the instructions
Check the k8s dashboard

## Before deploying anything
Be sure to add first your `PersistentVolumes` and thier `Claims`. <br/>
And then deploy all `ConfigMaps` and `Secrets`.
And then all your `Deployments`

# FAQ
## Why does microk8s not start after reboot?
Try to connect the computer to an internet connection and try again

## Permission errors on NFS
Don't forget to make the wanted folders before you deploy. <br/>
If the folder exists make sure you changed the ownership to - `nobody: nogroup`, <br/>
and the folder permissions to - `777`

## Self-Singed certified regsitry errors
Create a ssl certificate, it will create two files: `.crt` file and `.key`. You can use [Docker docs](https://docs.docker.com/registry/insecure/#use-self-signed-certificates)

1. Place them both in the volume of the Pod `/certs/domain.crt` and `/certs/domain.key`
2. Add the path of them to the environment variables of the Pod

### In each node of the cluster, Do like the following:
### Setup the docker daemon
1. Create a directory in this path `/etc/docker/certs.d/<host-name | ip-address>:<exposed-port>`
2. Place the certificate created above inside it with the name - `ca.crt`
3. Don't forget to restart the docker service
4. Check the docker enable to or push from this registry

#### Make Microk8s to support it (v1.23)
1. Place the `.crt` in `/usr/local/share/ca-certificates`
2. Run `sudo update-ca-certificates`
3. And check whether the certificate added successfully to `/etc/ssl/certs/ca-certificates.crt` file
4. Create a directory in this path `/var/snap/microk8s/current/args/certs.d/<host-name | ip-address>:<exposed-port>`
5. Create inside it a file named `hosts.toml`, you can find the content [here](https://microk8s.io/docs/registry-private) (or in the `/var/snap/microk8s/current/args/certs.d/localhost:32000` in case it exists)

Eventually, don't forget to restart the node