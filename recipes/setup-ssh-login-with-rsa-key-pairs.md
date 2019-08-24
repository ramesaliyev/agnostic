[Home](../README.md)

# Setup SSH Login with RSA Key Pairs

Easiest to do this is using `ssh-copy-id`

    ssh-copy-id <username>@<ip-address-or-hostname>

Use `-i` flag to specify `identity_file` if you dont want to use default one.

    ssh-copy-id -i <identity-file-path> <username>@<ip-address-or-hostname>

Example

    ssh-copy-id -i ~/.ssh/hetzner_rsa.pub ra@ra-vm1

$>

    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/ramesaliyev/.ssh/hetzner_rsa.pub"
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    ra@ra-vm1's password:

    // Enter user password

    Number of key(s) added:        1

    Now try logging into the machine, with:   "ssh 'ra@ra-vm1'"
    and check to make sure that only the key(s) you wanted were added.