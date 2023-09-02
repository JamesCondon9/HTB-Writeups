# HackTheBox - Keeper

Welcome to my writeup for the HTB room - Keeper!\
Obviously - MAJOR spoilers ahead. I recommend attempting this room by yourself and only seeking writeups if you are super duper stuck as otherwise you'll be hindering your chance to understand what's actually going on :)

**Difficulty: Easy**

**Point Value: 20**

---


Firstly, I set the IP environment variable for convinience.
```
export IP=<machine-ip>
```

Next, we run our default nmap scan.
```
nmap $IP -sC -sV --min-rate=1000
```
**[-sC]** - *Runs default nmap script that gives us extra information about services*\
**[-sV]** - *Discover service version types*\
**[--min-rate=1000]** - *Send at least 1000 packets per second*\

We get in return a relatively limited response that only registers ports `22` and `80` open.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3539d439404b1f6186dd7c37bb4b989e (ECDSA)
|_  256 1ae972be8bb105d5effedd80d8efc066 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Let's begin by enumerating the HTTP service using firefox.
Here is the page we're greeted with this simple webpage:

![1st Screenshot of Box](https://cdn.discordapp.com/attachments/704802474397008023/1147272499558617179/Keeper1.png)

The included hyperlink refers us to `http://tickets.keeper.htb/rt/`

As I'm not using the Pwnbox, this link leads nowhere as my computer doesn't know what to do with this link. I'll have to manually edit my **/etc/hosts** file and append the following:

``` 
<machine-ip> keeper.htb
<machine-ip> tickets.keeper.htb
```
Now we can acess the website as we'd expect. We're greeted with a simple login page that uses RT 4.4.4 from Best Practical.

![2nd Screenshot of Box](https://cdn.discordapp.com/attachments/704802474397008023/1147272507301314620/Keeper2.png)

After (a heck of a lot of) exploit researching, I came to a dead end. I found some potential for SQL injection exploits with older versions of the software but couldn't crack it for this one.
Eventually I came to my senses and tried the default credentials for Request Tracker. Here is a snapshot of the live playback of my reaction in this moment:

![Tyler Mugshot](https://cdn.discordapp.com/attachments/704802474397008023/1147274323334267061/tyler.png)

Sometimes the best solution is the easiest one. At least I've learnt my lesson.
Once authenticated, we can navigate the UI to reach a ticket titled 'Issue with Keepass Client on Windows'.

![3rd Screenshot of Box](https://cdn.discordapp.com/attachments/704802474397008023/1147272517342482472/Keeper3.png)

Google tells us that KeePass is a password manager which our friend Lise (username lnorgaard) has been having some difficulties with.


\>:)


We find that Lise has saved the crash dump of the program on her home directory. With our newfound incentive, let's try hack into their account!
Under *Admin* > *Users* on the website ribbon, we see a list of users on the system (including ourselves, root).
Expanding lnorgaard, we discover a Unix Login *lnorgaard* and password *Welcome2023!*

Using these as our SSH credentials...
```
ssh $IP -l lnorgaard
lnorgaard@<machine-ip>'s password:Welcome2023!
```
... promptly grants us access to the service.
`cat user.txt` reveals to us our user flag.

Weirdly, there seems to be some tools included on the server for us already.
The test.py file makes reference to a **{CVE-2023-32784}** which will allow us to access the master password from the aforementioned crash dump file that's also included in this directory.
`python3 test.py KeePassDumpFull.dmp` yields some potential master passwords (dots represent o with strike through them).

![4th Screenshot of Box](https://cdn.discordapp.com/attachments/704802474397008023/1147272532840435873/Keeper4.png)

Upon googling dgr●d med fl●de, I was autocorrected by google to rødgrød med fløde. Nice, this is our password.
I use `scp lnorgaard@<machine-ip>:passcodes.kdbx ~/Downloads` so I can use keepassx to open this (I can't install this program on the target, I have more access if I just download it).

Then, using keepassx and the password 'rødgrød med fløde', we find the root password: 'F4><3K0nd!'
![5th Screenshot of Box](https://cdn.discordapp.com/attachments/704802474397008023/1147272542575415468/Keeper5.png)

Now to log into the SSH service as root
`ssh root@<machine-ip>`
... but strangely, our password does not work.

Instead let's try and use the PPK file that is included in the notes section.
.ppk files are a PuTTY version of your typical ssh key. We can convert this by typing
`puttygen ppkkey.ppk -O private-openssh -o id_rsa` [^1]

[^1]: If you're having problems at this step, I was too because of strange version differences but this article helped me: [https://medium.com/@arslion/convert-ppk-version-3-to-ssh-private-public-keys-pem-on-linux-ubuntu-4bf2c8db1ef2]

Anyway, we can now ssh in as root to receive our reward. 

`ssh root@<machine-ip> -i id_rsa`

BANG! We're in. `cat root.txt` and now we win.

Thanks for reading my writeup! I still have a long way to come when learning in this field, but I appreciate you following along :D
\- James
