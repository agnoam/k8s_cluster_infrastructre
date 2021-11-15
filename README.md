This is cluster setup minimal instructions with `microk8s` and attached NFS storage.
First of all install [MicroK8s](https://microk8s.io/docs) on all of the computers in the cluster.

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
* Add networking settings in `/etc/exports` (example: `<path_to_dir> <static_ip_of_computer>/24(rw,sync,no_wdelay,root_squash,insecure,no_subtree_check,fsid=0)`)

More info in [install nfs server on ubuntu](https://www.tecmint.com/install-nfs-server-on-ubuntu/)

Run on one of the computers the command `microk8s dashboard-proxy` and follow the instructions
Check the k8s dashboard