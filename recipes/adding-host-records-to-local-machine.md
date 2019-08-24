[Home](../README.md)

# Adding Host Records to Local Machine

On your local machine open `/etc/hosts` file with a text editor.

    sudo vi /etc/hosts

Then add your host records to the end of it;

    <ip-address> <hostname>

Example;

    116.xxx.xxx.xxx ra-vm1
    116.xxx.xxx.xxx ra-vm2
    116.xxx.xxx.xxx ra-vm3

    // For easy usage with browsers.
    116.xxx.xxx.xxx ravm1.com
    116.xxx.xxx.xxx ravm2.com
    116.xxx.xxx.xxx ravm3.com