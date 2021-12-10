---
title: Set up an automatic SSH login on Linux
date: 2021-12-10 00:00:00 +0100
categories: [Linux, security]
tags: [linux, ssh]
---

Suppose you want to connect to/from a remote Linux machine (that might as well be a virtual machine) via SSH, but without having to type in the password each time.

In the following, I will be referring to:
* the `main` machine: the machine from which you want to SSH to the remote machine without typing in the password;
* the `remote` machine: the machine you are SSH-ing into from the main machine.

## Create a pair of SSH keys

In order to log in via SSH without typing in the password, the remote machine only needs to know the main machine via its public SSH key.

From the `main` machine, generate a SSH key pair to use for logging  in to the `remote` machine:

```
ssh-keygen -t rsa
```

Save the key pair as `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`.

Do **NOT** provide a passphrase if the intention is to use this automatic login for automated tasks (e.g. for scripts where SSH is used indirectly, such as when copying files or pushing to a git repository).

## Make the remote know the main machine

This is done by copying the public key of the `main` machine to the `remote` machine.

If the `~/.ssh` directory does not exist on `remote`, create it first. Then append the public key that was generated during the previous step to the list of known hosts for `remote`. From `main`:

```
cat ~/.ssh/id_rsa.pub | ssh user@remote 'cat >> ~/.ssh/authorized_keys'
```

## Test the automatic login

Now, from `main`, you should be able to SSH to the `remote` machine without having to type in the password:

```
ssh user@remote
```

## Caveat

Creating a SSH key pair without a passphrase means the private key remains unencrypted. If somebody has access to the `main` machine or manages to exploit a vulnerability and gains access to it, the private SSH key is just out there in plain sight.