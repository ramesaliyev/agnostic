[Home](../README.md)

# Fixing Node Internal IP

Check if the Internal IP address of `master-node` is correct.

    kubectl get nodes -o wide

Right now our internal address is wrong, because its pointing to our public IP.

    NAME                 STATUS   ROLES    AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
    ra-vm2-master-node   Ready    master   26m   v1.15.3   116.xxx.xxx.xxx   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://18.9.8

We need to fix this. Open the `kubelet` service configuration;

    sudo vi /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

Add `--node-ip=<private-node-ip>` to end of `ExecStart=` line.

    . . .
    EnvironmentFile=-/etc/default/kubelet
    ExecStart=
    ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --node-ip 10.0.0.x

Save and exit.

Restart kubelet.

    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

If you check now, you will see our nodes pointing correct Internal IPs.

    kubectl get nodes -o wide

$>

    NAME                 STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
    ra-vm2-master-node   Ready    master   36m   v1.15.3   10.0.0.x      <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64   docker://18.9.8

Yes they are.