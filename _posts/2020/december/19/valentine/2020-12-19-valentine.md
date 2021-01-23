---
layout: post
title: Hack The Box - Valentine
categories: ctf, htb
permalink: /htb-valentine/
published: true
---

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/1-1.png)



### Start of Recon

As always we start with nmap. Looking at the results we don't really see interesting stuff righ tof the bat. We have SSH on port 22, HTTP on 80 and HTTPS on 443. But if you look deep on the nmap scan and look at the versions, we can notice something a bit off. “OpenSSH 5.9" is probably an old version of Ubuntu server. It's also backup up by HTTP 2.2.22. The Ubuntu version is Precise Pangolin, release in 2012.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/2-1.png)



### Running vulnerability scripts in nmap to discover heartbleed + Going to the HTTP page to see what it looks like

Since it's an old release of Ubuntu, let's run a vulnerability scan against it. In the meantime let's check the website. A big old picture pops out and this is the heartbleed logo which is a vulnerability discovered when Precise Pangolin was end of life. Chances are we want to exploit heartbleed on this box.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/4-1.png)


The below nmap script came back with indeed a heartbleed but also a MITM vulnerability.
```bash
nmap --script vuln 10.10.10.79
```

The sslyze command is a cool to confirm our nmap script. But let's move on and exploit heartbleed.
![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/4-2.png)



### Begin of Heartbleed - Grabbing Python Module + Explaining Heartbleed -- XKCD ftw

We could use metasploit but it's always more fun to do it by hand. I think [xkcd](https://xkcd.com/) does a good job of explaining [how the exploit Heartbleed works](https://xkcd.com/1354/).
Typing python heartbleed.py to learn how to run this, looks we can use -x to tell us how many bytes are in the output easility, then 10.10.10.79. It stops at 4000 hex.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/6-1.png)


After trying four times there wasn't much information. We tried running the following command to see if there was any change but nope.

```bash
for i n $[seq 0 100); do python heartbleed.py -x 10.10.10.79; done
```

We have to look into the script. Let's modify the payload lenght to maximum so we don't miss anything.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/6-2.png)


Running this 100 times for good mesure.

```bash
python heartbleed.py -n 100 10.10.10.79
```


### Finding an encrypted SSH key on the server

While the python script is running let's check our directories. We have omg which is just the pop image we saw earlier.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/7-1.png)


The /dev folder on the other hand, contained some notes and this hype_key. It might be ASCII.
![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/7-2.png)


And it is! We have an encrypted RSA key. Let's save the key under hype.key.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/7-3.png)



### Examining heartbleed output to discover SSH password

Now let's go back to heartbleed. We have someone running a decode.php and putting in the string “aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==” several times.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/8-1.png)


That string is base64. So if we run the following command we get “heartbleedbelievethehype”

```bash
echo -n aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg== | base64 -d 
```

If we look at the key we just saved, hype.key, we have an encrypted SSH key. Let's chmod 600 hype.key so we can use it. So let's try to ssh into it with hype as user because that's what the server called it, hype_key. And of course the password. We are now live at Valentine.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/8-2.png)



### SSH as low priv user returned

On the desktop we find user.txt file. It is 33 characters which is 32 and a line break so that's our MD5sum of the user.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/9-1.png)


The very first thing we want to do is run a linux enumeration script. We can now inspect the box more closely.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/9-2.png)



### Finding a writable tmux socket to hijack session and find a root shell

First look at the bash_history file. We see tmux -S on dev_sess. 

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/10-1.png)


Running ```ps -ef  | grep root``` we can see all of root processes. We can see that root is running tmux.

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/10-2.png)


This socket file is owned by root but... the group owner is hype which is us. And that is read/write (rw).

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/10-3.png)


It means that if we enter tmux -S /.devs/dev_sess we can hop in root's tmux session. The box is ours!

![_config.yml]({{ site.baseurl }}/images/2020/december/19/valentine/10-4.png)


