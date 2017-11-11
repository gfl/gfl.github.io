---
layout: post
title:  "pass - Password management made easy"
author: Gracia Fernandez
date:   2017-11-11 19:35:47 +0000
categories: pass password management security
---

Overview
========

Sharing secrets, passwords and sensible information in a team is a recurrent challenge. Doing it via email, chats or pen-drives is both rudimentary and insecure by nature. A better approach are password managers.  

In this article, I am going to describe how to use [pass](https://www.passwordstore.org/), which is my preferred password store due to the following features:

* Integration with Git, allowing sharing and control version.
* Simple command line user interface.
* 3rd party UI and integrations: mobile, chrome, firefox…
* Standard for any Unix like system.
* Information stored in a tree-like hierarchy of folders.
* Open Source project (licensed under GPLv2+) with a very active community.

The main idea of pass is to store the passwords in gpg2 encrypted files organized into meaningful folder hierarchies within a git repository, which can be easily shared across the team.

Using pass
==========

If the password store is already initialised with your personal GPG key, 
you can clone the pass git repo and start using it:

```
$ pass clone git@gitserver.company.com/team/password-store ~/.password-store

```

Now you can read, create or delete keys

```
$ pass
Password Store
├── users
|   ├── test_user1
|   └── test_user2
└── aws
    └── 12345
          └── some@email.com
$ pass -c aws/12345/some@email.com
Copied aws/12345/some@email.com to clipboard. Will clear in 45 seconds.
$ pass generate Amazon/some@email.com 21 
$ pass rm users/test_user2
```

After changing the password store, you can push your changes:

```
$ pass git push
```

Installation
============

To install pass in your Mac you will only need to run:

```bash
$ brew install pass gnupg
```

If you are using Linux or the command above didn’t work in your Mac find below additional links with further information:
* http://zx2c4.com/projects/password-store/#download
* http://www.stackednotion.com/blog/2012/09/10/setting-up-pass-on-os-x/

Once your pass is installed in your system you can start using it. 

Setting up pass
===============

To use pass each member of the team has to create their own gpg2 keys, which will be used to encrypt the passwords in the password store.
 
Generate your own gpg keys
--------------------------

To generate your GPG key pair in Mac do the following:

```bash
$ gpg --gen-key
```

Once created, each member of the team needs to share their public keys with the others. You can do this in many ways. For example, you can publish your public key in a key server. In this example I've used the MIT PGP Public Key Server https://pgp.mit.edu/:

```bash
$ gpg --list-secret-keys --keyid-format LONG
...
/Users/gfl/.gnupg/pubring.gpg
----------------------------------
sec   rsa2048/FCF67125 2017-11-11 [SC] [expires: 2019-11-11]
uid                 [ultimate] Gfl Pass <graciafdez@gmail.com>
ssb   rsa2048/234D1E32 2017-11-11 [E] [expires: 2019-11-11]
$ gpg --keyserver pgp.mit.edu --send-key FCF67125
```

Importing the keys of your team members
----------------------------------------

To be able to generate or update the keys in the store, everybody needs to import their trustee public keys in their key store and sign them as follows:

```bash
$ gpg --keyserver pgp.mit.edu --recv-keys 0x12345678 0x23456789 0x34567890 ...
$ gpg --sign-key 0x12345678
$ gpg --sign-key 0x23456789
$ gpg --sign-key 0x34567890
...
```

As this might be tedious, you can also create a utility script to automatically import all keys from the `.gpg-id` file.

Initialise the pass store with new the gpg keys
------------------------------------------

Once you have all the keys of your team, you can initialise the password store. A new repository is just a new git rep.

This has to be done only once by only one of the members of the team:

```bash
$ pass init FCF67125 414d191e ... # all the team keys
$ pass git init
$ pass git remote add origin git@gitserver.company.com/team/password-store
$ pass git push -u --all
```

This are actually very simple operations
 * Creates the directory `~/.password-store`
 * Creates a file with `~/.password-store/.gpg-id` one line per key.
 * Performs a `git init` in `~/.password-store`

Add or remove a new team member
-------------------------------

To do so, you only need to rerun the command `pass init ...` with all the valid keys.

