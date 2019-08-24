[Home](../README.md)

# Cluster Initialization

## Initialize Cluster

In this step, we will initialize Kubernetes on the `master-node` node. Run all commands in this stage only on the `master-node` VM.

Get kubernetes version.

    sudo kubeadm version

$>

    kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:11:18Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}

Answer is `1.15.3` in this case.

Initialize the Kubernetes cluster using the kubeadm command below.

    sudo kubeadm init --pod-network-cidr=10.202.10.0/16 --apiserver-advertise-address=10.0.0.3 --apiserver-cert-extra-sans=10.8.0.1 --kubernetes-version "1.15.3"

> In case of getting `ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables` try `sudo modprobe br_netfilter`.

Note:
- `--apiserver-advertise-address=` determines which IP address Kubernetes should advertise its API server on. In this case its private network ip of our `master-node` VM.
- `--pod-network-cidr=` specify the range of IP addresses for the pod network. Change the range IP address to the what the network plugin wants. Use `10.244.0.0/16` if you gonna deploy flannel network. We are going to use `calico` so its okay to choose what we like.
- `--apiserver-cert-extra-sans` extra IPs that you want to connect from to kubernetes api. This is going to be our VPN network in future.
- `--ignore-preflight-errors=NumCPU` add this option only if your VMs has less than 2vCPU.

After the command output will be;

    [init] Using Kubernetes version: v1.15.3
    [preflight] Running pre-flight checks
    . . .
    Your Kubernetes control-plane has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

    Then you can join any number of worker nodes by running the following on each as root:

    kubeadm join 10.0.0.3:6443 --token 6p4kc7.0icgjrgj0shr7yid \
        --discovery-token-ca-cert-hash sha256:d3af0f24e1afba7345a1849aca104f1152103e75161126c37fb2d9f1b3e3116b

Copy the `kubeadm join ... ... ...` command to your text editor. The command will be used to register new worker nodes to the kubernetes cluster. Or better save this whole part after `To start using your cluster...` into your home folder.

    vi ~/k8s-kubeadm-join.txt

Now in order to use Kubernetes, we need to run some commands as shown in the result.

Create new `.kube` configuration directory and copy the configuration `admin.conf` from `/etc/kubernetes` directory.

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Now your cluster initialized, you can check the nodes;

    kubectl get nodes

But you will see the only node we have; the `master-node` is not ready yet. This is because we need to deploy a pod network to the cluster. We're going to use `calico` for this job.

## Deploying Calico the POD Network

Install `wget`;

    sudo yum install wget

Download manifest

    mkdir -p $HOME/.calico-config
    wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml -O $HOME/.calico-config/calico.yaml

Open it;

    vi $HOME/.calico-config/calico.yaml

Change `192.168.0.0/16` with our Pod CIDR `10.202.0.0/16`

    - name: CALICO_IPV4POOL_CIDR
      value: "10.202.0.0/16"

And add right under this config;

    - name: IP_AUTODETECTION_METHOD
        value: "interface=eth1"
    - name: IP6_AUTODETECTION_METHOD
        value: "interface=eth1"

**Here `eth1` is the interface name of our private network (`10.0.0.0/24`)**

Save and exit.

Apply manifest.

    kubectl apply -f $HOME/.calico-config/calico.yaml

$>

    configmap/calico-config created
    customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
    customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
    clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
    clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
    clusterrole.rbac.authorization.k8s.io/calico-node created
    clusterrolebinding.rbac.authorization.k8s.io/calico-node created
    daemonset.apps/calico-node created
    serviceaccount/calico-node created
    deployment.apps/calico-kube-controllers created
    serviceaccount/calico-kube-controllers created

Now check the nodes again;

    kubectl get nodes

You will see that our cluster is ready;

    NAME                 STATUS   ROLES    AGE   VERSION
    ra-vm2-master-node   Ready    master   14m   v1.15.3

Remove the taints on the master so that you can schedule pods on it.

    kubectl taint nodes --all node-role.kubernetes.io/master-

$>

    node/ra-vm2-master-node untainted

Confirm that you now have a node in your cluster with the following command.

    kubectl get nodes -o wide

Check the `IPv4Address` of the master node.

    kubectl describe node ra-vm2-master-node

$>

    [ra@ra-vm2-master-node ~]$ kubectl describe node ra-vm2-master-node
    Name:               ra-vm2-master-node
    Roles:              master
    Labels:             beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/os=linux
                        kubernetes.io/arch=amd64
                        kubernetes.io/hostname=ra-vm2-master-node
                        kubernetes.io/os=linux
                        node-role.kubernetes.io/master=
    Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                        node.alpha.kubernetes.io/ttl: 0
                        projectcalico.org/IPv4Address: 10.0.0.3/32
                        projectcalico.org/IPv4IPIPTunnelAddr: 10.202.217.0
                        volumes.kubernetes.io/controller-managed-attach-detach: true

If the `projectcalico.org/IPv4Address` value other than your private network IP. You have a trouble. `IP_AUTODETECTION_METHOD` variable we introduce above should've fixed this.

You can fix it temporarily by command below but its gonna reset after you restart calico pods.
You can try by;

    kubectl annotate node ra-vm2-master-node --overwrite projectcalico.org/IPv4Address=10.0.0.3

Restart calico pods

    kubectl delete pods --selector=k8s-app=calico-node -n kube-system

Check if the IP address of master node same;

    kubectl describe node ra-vm2-master-node

So this is fucked up and we dont have a solution. But if everything looks good, continue.

## System Check

Wait for a minute and then check kubernetes node and pods using commands below.

    kubectl get nodes
    kubectl get pods --all-namespaces

And you will get the `ra-vm2-master-node` node is running as a `master` cluster with status `ready`, and all `kube-system` pods that are needed for the cluster is up and running.

    NAMESPACE     NAME                                         READY   STATUS    RESTARTS   AGE
    kube-system   calico-kube-controllers-65b8787765-gbjgd     1/1     Running   0          11m
    kube-system   calico-node-q4259                            1/1     Running   0          11m
    kube-system   coredns-5c98db65d4-2v4t5                     1/1     Running   0          22m
    kube-system   coredns-5c98db65d4-wgdj2                     1/1     Running   0          22m
    kube-system   etcd-ra-vm2-master-node                      1/1     Running   0          21m
    kube-system   kube-apiserver-ra-vm2-master-node            1/1     Running   0          21m
    kube-system   kube-controller-manager-ra-vm2-master-node   1/1     Running   0          22m
    kube-system   kube-proxy-9w9n8                             1/1     Running   0          22m
    kube-system   kube-scheduler-ra-vm2-master-node            1/1     Running   0          22m

Now we're going to check if internal IP of our node is correct, refer to [Fixing Node Internal IP](./fixing-node-internal-ip.md) to check and fix it if necessary. Also you need to repeat these steps after every node you add to the cluster.

Now our cluster is ready to serve!