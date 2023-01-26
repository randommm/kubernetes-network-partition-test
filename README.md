Here we play a little with the effects of a network partition on k3s cluster with 2 or 3 control-plane nodes.

Prerequisites
==============
A machine with around 8GB of RAM and around 10G of free disk space.

Install multipass, with snap it's as simple as:

`sudo snap install multipass`

Installing first K3S node
============================

`multipass launch --cpus 2 --mem 1G --disk 10G --name server-1`

`multipass exec server-1 -- bash -c "curl -sfL https://get.k3s.io | sh -s - server --cluster-init"`

`multipass exec server-1 -- sudo k3s kubectl get nodes`


Installing an agent node
==========================================

`multipass launch --cpus 2 --mem 1G --disk 10G --name agent-1`

`SERVER_1_IP=$(multipass info server-1 | grep IPv4 | awk '{print $2}' | tr -d "\r\n")`

`K3S_TOKEN=$(multipass exec server-1 -- bash -c "sudo cat /var/lib/rancher/k3s/server/node-token")`

`multipass exec agent-1 -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=https://$SERVER_1_IP:6443 K3S_TOKEN=$K3S_TOKEN sh -"`

Installing a second server
==========================================

`multipass launch --cpus 2 --mem 1G --disk 10G --name server-2`

`multipass exec server-2 -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=https://$SERVER_1_IP:6443 K3S_TOKEN=$K3S_TOKEN sh -s - server"`

Deploy something
=================

`multipass exec server-1 -- sudo k3s kubectl create deployment nginx-deployment --image=nginx --replicas=2`

`multipass exec server-1 -- sudo k3s kubectl kubectl get pods -o wide`

`multipass exec server-2 -- sudo k3s kubectl scale deployment/nginx-deployment --replicas=4'`

`multipass exec server-1 -- sudo k3s kubectl get nodes`

Causing a network partition
=============================

`multipass exec server-2 -- sudo killall -STOP k3s-server`

`multipass exec server-1 -- sudo k3s kubectl get nodes`

`multipass exec server-2 -- sudo killall -CONT k3s-server`

`multipass exec server-1 -- sudo k3s kubectl get nodes`

Adding a third server
======================

`multipass launch --cpus 2 --mem 1G --disk 10G --name server-3`

`multipass exec server-3 -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=https://$SERVER_1_IP:6443 K3S_TOKEN=$K3S_TOKEN sh -s - server "`

Now let's cause a network partition again:

`multipass exec server-2 -- sudo killall -STOP k3s-server`

If you keep running the command below, you will notice that the server will keep answering your requests

`multipass exec server-1 -- sudo k3s kubectl get nodes`

`multipass exec server-1 -- sudo k3s kubectl get pods -o wide`



`multipass exec server-2 -- sudo killall -CONT k3s-server`

`multipass exec server-1 -- sudo k3s kubectl get nodes`

`multipass exec server-1 -- sudo k3s kubectl get pods -o wide`
