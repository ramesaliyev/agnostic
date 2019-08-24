[Home](../README.md)

# Disabling SSH Login with Password

> If you are going to hide servers behind VPN, this may not be necessary.

Open the file:

    sudo vi /etc/ssh/sshd_config

Inside of the file, search for the PasswordAuthentication directive. If it is commented out, uncomment it. Set it to "no" to disable password logins:

    PasswordAuthentication no

After you have made the change, save and close the file. To implement the changes, you should restart the SSH service.

    sudo systemctl restart sshd.service

Now, all accounts on the system will be unable to login with SSH using passwords.