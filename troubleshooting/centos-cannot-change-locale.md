[Home](../README.md)

# CentOS Cannot Change Locale

Error at login:

    -bash: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory

Solution;

    sudo vi /etc/environment

Add these lines;

    LANG=en_US.utf-8
    LC_ALL=en_US.utf-8