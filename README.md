#Ch-roomebook

The default Chromebook's operative system is Chrome OS, that can install
practically just web apps from the chrome web store. There is the possibility
to start another system in a chroot environment, this is well explained in the
[crouton project](https://github.com/dnschneid/crouton). The system launched
inside the chroot use the Linux kernel loaded with chrome OS and with this
trick we will have always two loaded systems. The problem about this 
solution is that there are many kernel modules that are not compiled 
in the kernel version of chrome OS.

I will put in this repository different solutions to solve common (and not) 
problems that I found inside the chrooted operative system. 

## Problems

### Mitmproxy

I tried to use [mitmproxy](http://mitmproxy.org/) to analyze the HTTP and HTTPS 
packets sent from my iPad, and obviously the [normal](http://blog.philippheckel.com/2013/07/01/how-to-use-mitmproxy-to-read-and-modify-https-traffic-of-your-phone/) [configuration](http://mitmproxy.org/doc/transparent.html) doesn't work inside
a chrooted system. The first thing that you need to install iptables

    # apt-get install iptables

then you can see (sudo iptables -L) that the default policy of the firewall is to drop every communication that is not specifically allowed. To make a transparent proxy you need to forward every packet that is not for your computer, so you have first to enable the kernel option for IP forwarding

    # sysctl -w net.ipv4.ip_forward=1

Then you need to accept the INPUT, OUTPUT and FORWARD target with

    # iptables -P FORWARD ACCEPT
    # iptables -P INPUT ACCEPT
    # iptables -P OUTPUT ACCEPT

The [official documentation](http://mitmproxy.org/doc/transparent/linux.html)of
*mitmproxy* suggest creating a rule to forward the traffic from the port 80
and 443 to 8080, that is the listening port of mitmproxy. I found out that the
REDIRECT rule for the NAT doesn't work, maybe is due to a [missing module in
the
kernel](http://stackoverflow.com/questions/24274337/iptables-error-iptables-no-chain-target-match-by-that-name).
My solution is to use the DNAT (Destination NAT) to change the packet destination to another port of the localhost.

    # iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j DNAT --to local_IP:8080

In this repository, there is a script that set all automatically. The usage is
 
    # ./setmitmproxy.sh --set

or to unset all

    # ./setmitmproxy.sh --unset

It needs to be executed as root, because to set the iptables rules or kernel options you need root privileges. 

### Samba connection
The kernel module to mount network file system is not compiled in the Chrome
OS kernel. There are different solutions to mount smb filesystem. 

[This](https://github.com/dnschneid/crouton/wiki/How-to-mount-network-shares-on-Chromebook-using-smbnetfs)
first solution (implemented from dnschneid) works if you have a chroot environment.
Another solution is explained [here](https://github.com/divx118/cifs/blob/master/README.md),
and don't need a chroot environment. I noticed that with the last chrome OS 
updates the last solutions doesn't work. 

I found out that with Ubuntu Trusty installed with crouton, you need only to
install smbnetfs

    # apt-get install smbnetfs

and to reboot the system or just exit and reenter from the chroot.
Finally, you should be able to mount the smb filesystem just by 
*connecting to a server* with *nautilus*.
(With this method you can't browse the network, but just connect to 
a specific server).

## To do

###Fix audio crackling

Check why the audio inside the chroot crackles. Maybe it's related to 
[how it is forwarded](https://github.com/dnschneid/crouton/wiki/Audio) 
to the default audio server of Chrome OS.