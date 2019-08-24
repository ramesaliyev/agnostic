[Home](../README.md)

# Disable Root User SSH Login

To do this, open the SSH daemon configuration file with root or sudo on your remote server.

    sudo vi /etc/ssh/sshd_config

Inside, search for a directive called PermitRootLogin. If it is commented, uncomment it. Change the value to "no":

    PermitRootLogin no

Save and close the file. To implement your changes, restart the SSH daemon.

    sudo systemctl restart sshd.service