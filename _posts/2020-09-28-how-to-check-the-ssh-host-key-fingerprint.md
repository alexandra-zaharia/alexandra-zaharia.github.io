---
title: How to check the SSH host key fingerprint
date: 2020-09-28 00:00:00 +0100
categories: [Linux, security]
tags: [linux, ssh]
---

When using [SSH][] to authenticate to a remote machine, password authentication can and should be replaced with SSH key pairs. 

To generate your public and private SSH key, run:

```
$ ssh-keygen -t rsa
Enter file in which to save the key (/home/alex/.ssh/id_rsa):
Created directory '/home/alex/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/alex/.ssh/id_rsa.
Your public key has been saved in /home/alex/.ssh/id_rsa.pub.
```

You shouldn't omit the passphrase, as it adds an extra layer of security. Your private key is `id_rsa` (and is optionally protected by a passphrase) and your private key is `id_rsa.pub`. Now you just copy over the public key to the remote machine. If the private key on your machine matches the public key on the remote machine, _voila_, you've got yourself a secure connection. 

All this is fine. However, there's something that bothers me to no end. I cannot count the number of times I've stumbled across [this][medium] kind of tutorial where authors just blatantly ignore checking whether the host they connect to is indeed the intended one. There's a message that is displayed the first time you connect to a SSH server, and there's a reason for this too. It goes something like this:

> The authenticity of host 'example.com (11.22.33.44)' can't be established. 
> ECDSA key fingerprint is \<some long string\>.
> Are you sure you want to continue connecting (yes/no)?

The reason behind this message is that the key fingerprint you're about to accept after due verification is added to your `~/.ssh/known_hosts` file. When this fingerprint changes, this might mean something changed on the server side, or that the server has been compromised through a [man-in-the-middle attack][mitm]. It is up to you to ensure you're always connecting to the machine you think you are connecting to.

So how do you check whether the server key fingerprint is valid? You either have access to it physically and run the command below, or you ask the sysadmin to do it for you:

```bash
for i in /etc/ssh/*pub ; do ssh-keygen -l -f $i ; done
```

One of those fingerprints must match the one that your client machine reports. Otherwise you should **not** proceed.

<!-- links -->
[SSH]: https://en.wikipedia.org/wiki/SSH_(Secure_Shell)
[medium]: https://medium.com/@jakewies/accessing-remote-machines-using-ssh-55a0fdf5e9d8
[mitm]: https://en.wikipedia.org/wiki/Man-in-the-middle_attack