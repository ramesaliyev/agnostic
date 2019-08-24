[Home](../README.md)

# Changing Port of SSH

> If you are going to hide servers behind VPN, this may not be necessary.

Open the file:

    sudo vi /etc/ssh/sshd_config

Locate the following line:

    # Port 22

Remove # and change 22 to your desired port number.

Restart the sshd service;

    sudo systemctl restart sshd.service
