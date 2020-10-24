# Provider Agnostic Scalable Server Environment Setup

> Unfortunately I couldn't find time to complete this walkthrough. But it still can be valuable.

What a long name. 😵

This documentation is for creating scalable server environment. The main goal is not using anything provider-specific such as managed services etc. Reason of this is to stay provider agnostic and cheap. This may not be best for production but good for learning. 👍🏿

This is more a walkthrough than a documentation, and is not meant to teach you anything, instead; we're assuming here that you already know the `what is` and `what for` the main pieces are, such as `VM`, `Cluster`, `SSH`, `Kubernetes`, `Firewall`, `RabbitMQ`, `PostgreSQL` etc, but having trouble at bringing them togehter. 🤷‍

Some things that you are going to read here is not written by me, i mostly just bring them togehter and make them complete by filling the gaps. But, believe me, there was great gaps which outputs lots of useless results. At first i tried to make reference to every resource i've used but at some point it become impossible for me. So sorry in advance for using contents without mentioning the authors. 😔

So this is a complete, tested, good for start server environment setup.

# Introduction and The Structure

Minimum setup is includes `3 VMs`;
  - First VM is for and only for databases;
    - We have only one database at the moment which is `PostgreSQL`.
    - In the future you can (and should) add another VMs for database replication/sharding.
  - Second VM is our `master` VM. We gonna use it as;
    - Kubernetes Master
    - Loadbalancer
    - Kubernetes Cluster Node
    - OpenVPN Server
  - Third VM is our `slave` VM. We gonna use it as;
    - Kubernetes Slave Cluster Node.

Point is; in the future, you gonna add as many `slave` nodes as you want. What we are describing here is the minimum initial setup, but if you want you can start more than one `slave` nodes. In that case you dont have to (and you better not to) use `master` node as a Kubernetes cluster node. Which means Kubernetes wont run PODs on it. But if you want to stay as cheap as possible for the initial setup its okay for now.

All VMs has to be `CentOS`, and mine is `CentOS 7.6` to be specific.

I'm using VMs (`droplets` in digitalocean jargon) that has `2GB RAM` and `2 CPUs` on [digitalocean](https://m.do.co/c/1181cc2e7c6b) for all of my VMs.

You need to have **private networking** enabled for VMs. In [digitalocean](https://m.do.co/c/1181cc2e7c6b) this is done by clicking a checkbox when creating the VMs. In some providers you need to manually create a private network from dashboard and assign it to VMs. Find a way and make sure that every VM has internet access and private networking.

Also remember to enable `backups` on first VM which has database on it.

I'm going to name my VMs as following;
  1. `ra-vm1-data-services`
  2. `ra-vm2-master-node`
  3. `ra-vm3-slave-node1`

We're going to run our `non-critical` `in-memory` databases such as `RabbitMQ` and `Redis` inside kubernetes cluster for scale them easily.

In walkthrough you can see my username in various places which is `ramesaliyev` and my home folder on my local machine is `/Users/ramesaliyev`.

# Step by Step Tutorial

1. First [create a RSA key pair](/recipes/creating-a-rsa-key-pair.md) on your local machine.
2. Create all infrastructure as described under [Architecture](#Architecture) title.
3. [Add hosts record](/recipes/adding-host-records-to-local-machine.md)
    - For all 3 VMs on your local machine.
      - Example;
        - `116.xxx.xxx.xxx ra-vm1`
        - `116.xxx.xxx.xxx ra-vm2`
        - `116.xxx.xxx.xxx ra-vm3`
        - `// For easy usage with browsers.`
        - `116.xxx.xxx.xxx ravm1.com`
        - `116.xxx.xxx.xxx ravm2.com`
        - `116.xxx.xxx.xxx ravm3.com`
    - For all 3 VMs on each of your VMs.
      - Remember to use private network IPs of VMs.
      - And use `127.0.0.1` for own host records.
      - Example;
        - `10.0.0.2 vm1 data-services`
        - `10.0.0.3 vm2 master-node`
        - `10.0.0.4 vm3 slave-node1`
4. [Login into all 3 VMs SSH](/recipes/logging-into-servers-ssh.md).
    - After first login terminal will prompt you to change password of `root` user. After changing the password logout and relogin to verify the changed password.
    - If password changing didn't prompt; refer to [Changing User Password Recipe](/recipes/changing-user-password.md) to change `root` password.
    - If you get `cannot change locale` error on login; refer to troubleshooting [CentOS Cannot Change Locale](/troubleshooting/centos-cannot-change-locale.md).
5. In all VMs make some adjustments for SSH;
    - [Setup SSH Session Idle Timeout Time](/recipes/setup-ssh-session-idle-timeout-time.md).
    - `Optional:` [colorize the terminal](/recipes/colorizing-the-terminal.md).
    - [Create a new user with sudo privileges](/recipes/creating-user-with-sudo-privileges.md).
    - For that user; [setup SSH login with RSA key pairs](/recipes/setup-ssh-login-with-rsa-key-pairs.md).
    - [Limit SSH users](/recipes/limiting-ssh-users.md) to newly created user.
    - [Disable Root User SSH Login](/recipes/disable-root-user-ssh-login.md)
6. Make preparations for kubernetes installation.
    - Check `hostname` of each VM, and [change them](/recipes/changing-hostnames.md) to be unique if they are same.
7. Kubernetes Up & Going
   - [Install Kubernetes](/kubernetes/installation.md) with all of its dependencies onto your `master` and `slave` VM.
   - `Optional:` Take a snapshot of all of your VMs and name it `initial`.
   - [Initialize kubernetes cluster](/kubernetes/cluster-initialization.md).
   - [Add worker nodes to the cluster](/kubernetes/adding-worker-nodes.md).
   - `Optional:` Take another snapshot of `vm2` and `vm3` VMs and name it `clustered`.
   - `Optional:` [Test kubernetes PODs and Services](/kubernetes/test-pods-and-services.md).
   - [Setup the Ingress](/kubernetes/ingress-setup.md).

# Notes

1. Following precautions may be taken for security but we're not gonna do those because we're going to secure our servers behind the VPN;
    - [Changing Port of SSH](/recipes/changing-port-of-ssh.md)
    - [Disabling SSH Login with Password](/recipes/disabling-ssh-login-with-password.md).

# Table of Contents

- [Cheatsheets](/cheatsheets)
  - [Kubernetes](/cheatsheets/k8s.md)
  - [UFW](/cheatsheets/ufw.md)
- [Recipes](/recipes)
  - [Creating a RSA Key Pair](/recipes/creating-a-rsa-key-pair.md)
  - [Adding Host Records to Local Machine](/recipes/adding-host-records-to-local-machine.md)
  - [Logging into Servers SSH](/recipes/logging-into-servers-ssh.md)
  - [Changing User Password](/recipes/changing-user-password.md)
  - [Setup SSH Session Idle Timeout Time](/recipes/setup-ssh-session-idle-timeout-time.md)
  - [Colorizing the Terminal](/recipes/colorizing-the-terminal.md)
  - [Creating User with Sudo Privileges](/recipes/creating-user-with-sudo-privileges.md)
  - [Setup SSH Login with RSA Key Pairs](/recipes/setup-ssh-login-with-rsa-key-pairs.md)
  - [Disabling SSH Login with Password](/recipes/disabling-ssh-login-with-password.md)
  - [Changing Port of SSH](/recipes/changing-port-of-ssh.md)
  - [Limiting SSH Users](/recipes/limiting-ssh-users.md)
  - [Disable Root User SSH Login](/recipes/disable-root-user-ssh-login.md)
  - [Changing Hostnames](/recipes/changing-hostnames.md)
  - [Copy File From and To Server](/recipes/copy-file-from-and-to-server.md)
- [Troubleshooting](/troubleshooting)
  - [CentOS Cannot Change Locale](/troubleshooting/centos-cannot-change-locale.md)
- [Kubernetes](/kubernetes)
  - [Installation](/kubernetes/installation.md)
  - [Cluster Initialization](/kubernetes/cluster-initialization.md)
  - [Fixing Node Internal IP](/kubernetes/fixing-node-internal-ip.md)
  - [Adding Worker Nodes](/kubernetes/adding-worker-nodes.md)
  - [Test PODs and Services](/kubernetes/test-pods-and-services.md)
  - [Ingress Setup](/kubernetes/ingress-setup.md)
