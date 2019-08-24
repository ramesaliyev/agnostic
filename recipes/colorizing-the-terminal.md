[Home](../README.md)

# Colorizing the Terminal

Command line prompt setting is store in a variable called `PS1`, we can check the current(default) setting using;

    echo $PS1

$>

    [\u@\h \W]\$

We can break the output into four parts which are :

1. `\u`; prints out the current login `username`, which is `root`.
2. `\h`; prints out the `hostname`, which is `ra-vm2-master-node`, as shown in hostname status. If we want to show the `FQDN`, we can change it to `\H`.
3. `\W`; prints out the current working directory, which is `~`, the home directory of login user. If we want to show the full working directory, we can change it to `\w`.
4. `\$`; prints `$` when user is login, `#` when root is login.

`$PS1` variable can be saved in `/etc/bashrc` which will affect all users or `/home/username/.bashrc` which will only affect that particular user.

Lets save this output as `PS1='[\u@\h \W]\$ '` at the bottom of `/etc/bashrc`. Please note that there is a space after the `$` sign, so there will be a space between the `$` or `#` sign and the cursor.

    sudo vi /etc/bashrc

Go to the end of the file and add;

    PS1='[\u@\h \W]\$ '

Re-login to check the result. Its same now.

Now let’s start adding colors to the prompt. The basic syntax of a color is `\[\e[xx;yy;zzm\]` where :

1. `\[\e[m\]` is the structure of the color syntax.
2. `xx;yy;zz` represents the `text formate`, `text color` and `text background color`. `xx`, `yy` and `zz` are unique numbers so you can put them in any order you wish.

A detailed color table is below:

| Text Format        | Text Color | Text Background Color |
|--------------------|------------|-----------------------|
| 0: normal text     | 30: Black  | 40: Black             |
| 1: bold            | 31: Red    | 41: Red               |
| 4: Underlined text | 32: Green  | 42: Green             |
|                    | 33: Yellow | 43: Yellow            |
|                    | 34: Blue   | 44: Blue              |
|                    | 35: Purple | 45: Purple            |
|                    | 36: Cyan   | 46: Cyan              |
|                    | 37: White  | 47: White             |

Since we want all text to be green we just gonna use two color identifier;

    PS1='\[\e[01;32m\][\u@\h \W]\$\[\e[0;37m\] '