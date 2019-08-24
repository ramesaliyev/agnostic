[Home](../README.md)

# Kubernetes Cheatsheet

## Debugging

Get a shell to the running Container:

    kubectl exec -it <name> -- /bin/bash

Get logs of container;

    kubectl logs <name>

# Namespaces

Get current context.

    kubectl config current-context

Create namespace.

    kubectl create ns <namespace>

Change the current default namespace.

    kubectl config set-context <context-name> --namespace <namespace>

    // or

    kubectl config set-context --current --namespace <namespace>

Check by getting current namespace

    kubectl config view | grep namespace

List namespaces

    kubectl get namespace

## Remove Node

### On Master

List the nodes and get the <node-name> you want to drain or (remove from cluster)

    kubectl get nodes

First drain the node

    kubectl drain <node-name>

You might have to ignore daemonsets and local-data in the machine

    kubectl drain <node-name> --ignore-daemonsets --delete-local-data

Finally delete the node

    kubectl delete node <node-name>

Get nodes again to verify node is deleted

    kubectl get nodes

**Now on every node apply `remove cluster` step.**

## Remove Cluster

    sudo kubeadm reset

    sudo systemctl stop kubelet
    sudo systemctl stop docker

    sudo rm -rf /var/lib/cni/
    sudo rm -rf /var/lib/kubelet/*
    sudo rm -rf /run/flannel
    sudo rm -rf /etc/cni/

    sudo brctl delbr cni0

Remember to take down network interfaces (check names)
Be careful to dont remove any other networks.

    sudo ifconfig cni0 down && sudo ip link delete cni0
    sudo ifconfig flannel.1 down && sudo ip link delete flannel.1

You may also reset iptables;

    iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X  # will reset iptables

Remove `.kube` folder under the home directory.

    rm -rf $HOME/.kube

Start back docker

    sudo systemctl start docker

## Renew Certificates

Check certificate expirations.

    sudo kubeadm alpha certs check-expiration

Renew manually

    sudo kubeadm alpha certs renew -h // for see names

    sudo kubeadm alpha certs renew admin.conf