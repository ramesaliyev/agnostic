[Home](../README.md)

# UFW Cheatsheet

To allow other connections you can do:

    sudo ufw allow http
    // or
    sudo ufw allow 80

    sudo ufw allow https
    // or
    sudo ufw allow 443

There are several others ways to allow other connections, aside from specifying a port or known service.

To allow specific port ranges:

    sudo ufw allow 6000:6007/tcp
    sudo ufw allow 6000:6007/udp

When specifying port ranges with UFW, you must specify the protocol (tcp or udp) that the rules should apply to. We haven't mentioned this before because not specifying the protocol automatically allows both protocols, which is OK in most cases.

To allow specific IP addresses:

    sudo ufw allow from 203.0.113.4

You can also specify a specific port that the IP address is allowed to connect to by adding `to any port` followed by the `port number`.

    sudo ufw allow from 203.0.113.4 to any port 22

To allow a subnet of IP addresses, you can do so using CIDR notation:

    sudo ufw allow from 203.0.113.0/24

Likewise, you may also specify the destination port that the subnet 203.0.113.0/24 is allowed to connect to. Again, we'll use port 22 (SSH) as an example:

    sudo ufw allow from 203.0.113.0/24 to any port 22

If you want to create a firewall rule that only applies to a specific network interface, you can do so by specifying "allow in on" followed by the name of the network interface.

You may want to look up your network interfaces before continuing. To do so, use this command:

    ip addr

This will output the network interface names. They are typically named something like eth0 or enp3s2.

So, if your server has a public network interface called eth0, you could allow HTTP traffic (port 80) to it with this command:

    sudo ufw allow in on eth0 to any port 80

Doing so would allow your server to receive HTTP requests from the public internet.

Or, if you want your MySQL database server (port 3306) to listen for connections on the private network interface eth1, for example, you could use this command:

    sudo ufw allow in on eth1 to any port 3306

This would allow other servers on your private network to connect to your MySQL database.

To allow all connections from specific network:

    sudo ufw allow in on eth1

To deny all connections from specific network:

    sudo ufw deny in on eth1

To deny connections:

For example, to deny HTTP connections, you could use this command:

    sudo ufw deny http

Or if you want to deny all connections from 203.0.113.4 you could use this command:

    sudo ufw deny from 203.0.113.4

Deleting rules:

Delete by rule number:

If you're using the rule number to delete firewall rules, the first thing you'll want to do is get a list of your firewall rules. The UFW status command has an option to display numbers next to each rule, as demonstrated here:

    sudo ufw status numbered

$>

    Numbered Output:
    Status: active

     To                         Action      From
     --                         ------      ----
    [ 1] 22                         ALLOW IN    15.15.15.0/24
    [ 2] 80                         ALLOW IN    Anywhere

If we decide that we want to delete rule 2, the one that allows port 80 (HTTP) connections, we can specify it in a UFW delete command like this:

    sudo ufw delete 2

This would show a confirmation prompt then delete rule 2, which allows HTTP connections. Note that if you have IPv6 enabled, you would want to delete the corresponding IPv6 rule as well.

Delete by actual rule:

The alternative to rule numbers is to specify the actual rule to delete. For example, if you want to remove the allow http rule, you could write it like this:

    sudo ufw delete allow http

    // or

    sudo ufw delete allow 80

This method will delete both IPv4 and IPv6 rules, if they exist.

Disabling the firewal:

    sudo ufw disable

Resetting the firewal:

    sudo ufw reset

---

Deny Connections to a Network Interface

    sudo ufw deny in on eth0 from 15.15.15.51

Allow Incoming SSH from Specific IP Address or Subnet

    sudo ufw allow from 15.15.15.0/24 to any port 22

Allow PostgreSQL to Specific Network Interface

    sudo ufw allow in on eth1 to any port 5432

Block Outgoing SMTP Mail

    sudo ufw deny out 25

Allow All Incoming SMTP

    sudo ufw allow 25

Check if loggin enabled

    sudo ufw status verbose

    // enable logging
    sudo ufw logging on

    // disable logging
    sudo ufw logging off

    // change logging level
    // level may be: off, low, medium, high, full
    sudo ufw logging <level>

    // check log files
    sudo ls /var/log/ufw*