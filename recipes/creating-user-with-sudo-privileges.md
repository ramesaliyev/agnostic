[Home](../README.md)

# Creating User with Sudo Privileges

Here we're going to use `ra` as a new user's name.

Add user:

    sudo adduser ra

Update password;

    sudo passwd ra

Add the user to the wheel group; (By default, on CentOS, members of the wheel group have sudo privileges.)

    sudo usermod -aG wheel ra

Test sudo access on new user account; (`su` means substitute user)

    su - ra

Try something with `sudo` command. For example, you can list the contents of the /root directory, which is normally only accessible to the root user.

    sudo ls -la /root