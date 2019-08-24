[Home](../README.md)

# Creating a RSA Key Pair

To generate an RSA key pair on your local computer, type:

    ssh-keygen -b 4096

$>

    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/ramesaliyev/.ssh/id_rsa):

This prompt allows you to choose the location to store your RSA private key. Press ENTER to leave this as the default, which will store them in the .ssh hidden directory in your user's home directory. Leaving the default location selected will allow your SSH client to find the keys automatically.

$>

    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:

The next prompt allows you to enter a passphrase of an arbitrary length to secure your private key. By default, you will have to enter any passphrase you set here every time you use the private key, as an additional security measure. Feel free to press ENTER to leave this blank if you do not want a passphrase. Keep in mind though that this will allow anyone who gains control of your private key to login to your servers.

If you changed the filename, you need to add your newly created key to `ssh-agent`;

    ssh-add ~/.ssh/<filename>

Then you can check your keys;

    ssh-add -l

After you create your server, you can add this SSH key to it so you can login without password.