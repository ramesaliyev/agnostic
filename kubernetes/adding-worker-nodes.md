[Home](../README.md)

# Adding Worker Nodes

In this step, we will add the worker node `ra-vm3-slave-node1` to the Kubernetes Cluster.

Connect to the `ra-vm3-slave-node1` server and run the kubeadm join command that you get from the cluster initialization.

    sudo kubeadm join 10.0.0.3:6443 --token 6p4kc7.0icgjrgj0shr7yid \
        --discovery-token-ca-cert-hash sha256:d3af0f24e1afba7345a1849aca104f1152103e75161126c37fb2d9f1b3e3116b

> In case of getting `ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables`
> try `sudo modprobe br_netfilter`.

$>

    [preflight] Running pre-flight checks
    [preflight] Reading configuration from the cluster...
    [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.15" ConfigMap in the kube-system namespace
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
    [kubelet-start] Activating the kubelet service
    [kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

    This node has joined the cluster:
    * Certificate signing request was sent to apiserver and a response was received.
    * The Kubelet was informed of the new secure connection details.

    Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

Wait for some minutes and back to the `ra-vm2-master-node` node master and check node status.

    kubectl get nodes

You will see the worker node `ra-vm3-slave-node1` is part of the Kubernetes Cluster.

    NAME                 STATUS   ROLES    AGE   VERSION
    ra-vm2-master-node   Ready    master   44m   v1.15.3
    ra-vm3-slave-node1   Ready    <none>   80s   v1.15.3

Now we're going to check if internal IP of our node is correct, refer to [Fixing Node Internal IP](./fixing-node-internal-ip.md) to check and fix it if necessary.