[Home](../README.md)

# Copy File From and To Server

    scp <source> <destination>

Examples;

    // From server.

    scp ra@ra-vm2:~/.kube/config ~/.kube/config-ramesaliyev

    // To server.

    scp ~/.kube/config-ramesaliyev ra@ra-vm2:~/.kube/config