[Home](../README.md)

# Limiting SSH Users

Open the file:

    sudo vi /etc/ssh/sshd_config

The first method of specifying the accounts that are allowed to login is using the `AllowUsers` directive. Search for the `AllowUsers` directive in the file. If one does not exist, create it anywhere. After the directive, list the user accounts that should be allowed to login through SSH:

    AllowUsers ra

Save and close the file. Restart the daemon to implement your changes.

    sudo systemctl restart sshd.service

Another way: If you are more comfortable with group management, you can use the AllowGroups directive instead. If this is the case, just add a single group that should be allowed SSH access (we will create this group and add members momentarily):
