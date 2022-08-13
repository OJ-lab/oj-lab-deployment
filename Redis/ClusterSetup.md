# Setup Redis Cluster

This document will show you how to deploy a redis cluster on to K8s.

It should be noticed that,
**some config in `.yaml` need to be correctly changed** according to real situation.
EX. `spec.nfs.server` should be changed to your own server IP
(And if you have more than one machine,
then you should have set more than one IP)

You've got all you need in this directory, `cd Redis` before you start.

## Step 1. Create NFS (Network File System)

This step can be slightly different in different operating system.
We'll provid a Ubuntu version for reference.

First edit `/etc/exports`,
add content like `/usr/local/k8s/redis/pv1 *(rw,sync,no_root_squash)`
to set the path you want to share.

Then run `systemctl restart rpcbind`, `systemctl restart nfs-kernel-server`
and `systemctl enable nfs-kernel-server`
(Make sure you have installed `rpcbind` and `nfs-kernel-server`).

## Step 2. Create K8s PV (Presistent Volume)

Check the content in `PersistentVolume.yaml`,
and make sure each block is set in right.
Run `kubectl create -f PersistentVolume.yaml`.

## Step 3. Prepare Redis configMap

Run `kubectl create configmap redis-conf --from-file=redis.conf`

## Step 4. Create headless service

Run `kubectl create -f HeadlessService.yaml`.

## Step 5. Add Redis contianer

Run `kubectl create -f Redis.yaml`

## Step 6. Set Redis cluster

Enter the Ubuntu container by `kubectl exec -it ubuntu -- /bin/bash`.

Download redis source `wget http://download.redis.io/releases/redis-7.0.3.tar.gz`
uncompress `tar -xvzf redis-7.0.3.tar.gz`
and make `cd redis-7.0.3 && make`.

Copy redis-cli to `/usr/local/bin` so than we can use it conveniently in command line:
`cp src/redis-cli /usr/local/bin/redis-cli`

Create cluster by:
``` sh
redis-cli --cluster create \
    `dig +short redis-app-0.redis-service.default.svc.cluster.local`:6379 \
    `dig +short redis-app-1.redis-service.default.svc.cluster.local`:6379 \
    `dig +short redis-app-2.redis-service.default.svc.cluster.local`:6379
```

Add salve by:
``` sh
redis-cli --cluster add-node \
    `dig +short redis-app-3.redis-service.default.svc.cluster.local`:6379 \
    `dig +short redis-app-0.redis-service.default.svc.cluster.local`:6379 \
    --cluster-slave --cluster-master-id 1a4c50db9833779be78cd6572cd2312974341570
```
You should do the same thing to another 2 redis node like the following command.
**The `--cluster-master-id` can be found in the `Create cluster step`**
(In case you forget this, you can also find them in your redis pod.
And the command you need is `kubectl exec -it redis-app-0 -- /bin/bash`, `redis-cli -c`,
these command will help you enter the redis-cli command line.
Then run `cluster nodes` to get node ID)