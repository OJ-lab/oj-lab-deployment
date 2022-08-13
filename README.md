# OJ Lab Deloyment

This repository is used mainly for recording deploying step of OJ Lab system.

You should perpare a K8s cluster firstm if you want try the following step out.

The most quickly approach is to use `minikube` on one machine,
and is great for testing.
Start minikube by `minikube start`, and add some node by `minikube node add`.
(If you meet `Too many open file` error, use `sysctl fs.inotify.max_user_watches=4194304` to fix it.)

# Tools

Adding a Ubuntu pod can be useful to help testing and setuping service.
Run `kubectl apply -f Ubuntu.yaml`.

## Instruction

- To build a redis cluster, see more in [Redis/ClusterSetup.md](Redis/ClusterSetup.md)