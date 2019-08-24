[Home](../README.md)

# Setup SSH Session Idle Timeout Time

There are two options related to ssh inactivity in `/etc/ssh/sshd_config` file:

    ClientAliveInterval
    ClientAliveCountMax

So the timeout value is calculated by multiplying `ClientAliveInterval` with `ClientAliveCountMax`.

There are 2 methods to configure the inactivity timeout. For example in this post we will configure an auto logout interval of 10 mins.

Open the file:

    sudo vi /etc/ssh/sshd_config

## Method 1

Configure the timeout value in the `/etc/ssh/sshd_config` file with below parameter values.

    ClientAliveInterval 5m          # 5 minutes
    ClientAliveCountMax 2           # 2 times

Restart the ssh service after setting the values.

    sudo systemctl restart sshd.service

This would make the session timeout in `10` minutes as the `ClientAliveCountMax` value is multiplied by the `ClientAliveInterval` value.

## Method 2

You can set the `ClientAliveCountMax` value to `0` and `ClientAliveInterval` value to `10m` to achieve the same thing.

    ClientAliveInterval 10m          # 10 minutes
    ClientAliveCountMax 0            # 0 times

Restart the ssh service after setting the values.

    sudo systemctl restart sshd.service

## Difference between method 1 and method 2

There’s a little difference between these two methods. For the first method, sshd will send messages, called Client Alive Messages here, through the encrypted channel to request a response from client if client is inactive for five minutes. The sshd daemon will send these messages max two times. If this threshold is reached while Client Alive Messages are being sent, sshd will disconnect the client.

But for the second method, sshd will not send client alive messages and terminate the session directly if client is inactive for 10 minutes.