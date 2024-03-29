---
layout: default
title: Dev Setup
nav_order: 1
parent: Guides
permalink: /guides/dev-setup
---

# RISC-V Development Environment Setup

## Overview

In this class we will be learning the RISC-V instruction set architecture. In order to do so we will work on both real and emulated RISC-V environments. For running on a real RISC-V machine, the CS Department has 5 BeagleV-Ahead boards:

[https://www.beagleboard.org/boards/beaglev-ahead](https://www.beagleboard.org/boards/beaglev-ahead)

You can also work in a emulated RISC-V environment using Qemu (https://www.qemu.org/). You will have two ways access RISC-V Ubuntu Linux: you can ssh into the CS BeagleV machines or you can run the RISC-V virtual machine locally using Qemu. This guide will help you set up access to both the remote BeagleV machines and a local RISC-V vm.

## Command Line

We will be working extensively from the shell (commmand line) in this class. See the [Shell Usage Guide](/guides/shell-usage) for an overview of working from the shell and common commands I frequently use.

To access the command line in macOS you can use the Terminal application or iTerm (a popular third party terminal program that is very powerful, [https://iterm2.com/](https://iterm2.com/)).

Linux also has a Terminal application for accessing the command line.

For Windows, I recommand you use Ubuntu Windows Subsystem for Linux (WSL):
- [Ubuntu WSL](https://apps.microsoft.com/store/detail/ubuntu-22042-lts/9PN20MSR04DW?hl=en-us&gl=us&rtc=1)
You can also get command line access with Gitbash, which comes with Git for Windows:
- [Git for Windows](https://gitforwindows.org)
- However, WSL will be better and easier for running the local RISC-V vm.

## ssh to stargate

We will be using ssh to access our remote RISC-V machines as well as our local RISC-V vm. From outside the CS network, you can access CS machines from stargate. You will need to install the USF VPN to access stargate outside the USF campus network ([USF VPN](https://myusf.usfca.edu/vpn)). To get to stargate type:

Note, all the commands below should be entered at the shell prompt, usually ```$``` (bash or sh), or ```%```.

```text
ssh <your_username>@stargate.cs.usfca.edu
```
Be sure to replace ```<your_username>``` with your USF/CS username, for example, I would type:

```text
% ssh benson@stargate.cs.usfca.edu
<You will see an ASCII art image and some informational text>
Last login: Thu Aug 24 00:09:00 2023 from c-73-92-205-177.hsd1.ca.comcast.net
[benson@stargate ~]$
```

Your default password on stargate is your full 8 digit USF ID number (CWID). If you changed your password and have forgotten it please email CS Support to ask them to reset your password on stargate:
- ```support@cs.usfca.edu```

If your password is still your default password, it is recommended that you change it with the ```passwd``` command:

```text
passwd
```

## Set up ssh keys for stargate

Typing your password every time you want to get to stargate will become annoying very quick. You can set up ssh keys so that you don't have to type your password ever single time. Also, typing passwords is not as secure as using keys. We will also setup ssh keys to access the Ubuntu RISC-V vm.

The basic idea is that we need to create a key pair that consists of a private key and a public key. We will put the public on the remote machines we want to access without typing a password.

Note that in the instructions below it is very important to double check two things:
- The computer that you sould be working on, that is your laptop or the remote machine.
- The directory that you should be in, i.e., ```~/.ssh```.
I will explain which computer and which directory you should be using. A common mistake is to execute commands on the wrong computer or in the wrong directory.

**ON YOUR COMPUTER**
- Go to your ```.ssh``` directory:

  ```text
  cd
  cd .ssh
  ```

- Create an ssh keypair (**in ~/.ssh on your computer**):

  ```
  ssh-keygen -t ed25519 -C "<your_username>@dons.usfca.edu" -f id_ed25519_cs631_2024s
  ```

  - Replace ```<your_username>``` with your USF/CS username.
  - This will ask you to enter a passphrase, which is like password. A longer sentence of English words is better than a short password with special characters. You need to remember this passphrase. You will be asked to enter it twice.
- Create a config file (**in ~/.ssh on your computer**):
  - Use an editor like ```nano```, ```vim```, or ```micro```

  ```text
  nano config
  ```

  - Add the following text:

  ```text
  IgnoreUnknown UseKeychain
  UseKeychain yes
  
  Host stargate
    HostName stargate.cs.usfca.edu
    AddKeysToAgent yes
    ForwardAgent yes
    IdentityFile ~/.ssh/id_ed25519_cs631_2024s
    User <your_username>
  ```

  - Be sure to replace ```<your_username>``` with your USF/CS username.
- Set the correct permissions of all files in ```~/.ssh```:

  ```text
  cd .ssh
  chmod 600 *
  ```

- Copy your public key to your ```.ssh``` directory on stargate:

  ```
  scp id_ed25519_cs631_2024s.pub stargate:.ssh
  ```

**ON stargate**
- ssh into stargate:

  ```text
  ssh stargate
  ```

  - At this point you still have to type your password

- Go to your ```.ssh``` directory:

  ```text
  cd .ssh
  ```

- Add your public key to the ```authorized_keys``` file.

  ```text
  cat id_ed25519_cs631_2024s.pub >> authorized_keys
  ```

  - This will create ```authorized_keys``` if it doesn't exist or it will add your new key to the end of ```authorized_keys``` if it does exist.
- Set the file permissions in your ```~/.ssh``` directory:

  ```text
  cd .ssh
  chmod 600 *
  ```

- Now exit back to your computer:

  ```text
  exit
  ```

**ON YOUR COMPUTER**
- ssh into stargate

  ```text
  ssh stargate
  ```

  - This time it should ask for your passphrase.
  - Now exit back to your computer:

  ```text
  exit
  ```

- ssh into stargate again

  ```text
  ssh stargate
  ```

  - This time it should not ask for your passphrase
  
- Note to Windows users
  - To get your ssh to not ask for a pasword you should follow the answer to this post:
  - [https://stackoverflow.com/questions/18880024/start-ssh-agent-on-login](https://stackoverflow.com/questions/18880024/start-ssh-agent-on-login)
  - You will need to add text to your ```~/.bash_profile``` in Ubuntu WSL.
  - After you do this you will need to exit the Ubuntu terminal and start a new one.

## Accessing the BeagleV machines

You can access the BeagleV machine from stargate:

```
ssh beagle
```

This will take you to one of the five BeagleV machines we have running in CS: beagle1, beagle2, beagle3, beagle4, and beagle5. Note that you have a shared home directory on stargate and all the beagle machines, so you ssh setup should work without changes.

To get your GitHub ssh key to propogate from stargate to the beagle machines you need to add the following to ```~/.ssh/config```:

```
Host *
    ForwardAgent yes
```

## Running the RISC-V vm on your computer

In addition to getting access to the remote RISC-V BeagleV machines you also need to set up a local RISC-V vm. This way you have two options for developing code in the class.

On your laptop, you need to install Qemu.

### Installing Qemu on macOS

The easiest way to install Qemu on macOS is to install homebrew:
- [https://brew.sh](https://brew.sh)
- Simple execute the following:

```text
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Once this is complete, you will need to add the homebrew path to your PATH

```text
cd
nano .zprofile
```

- Add the following text:

```text
export PATH=/opt/homebrew/bin:$PATH
```

- Now, exit your Terminal and start a new Terminal
- This will activate the new PATH and allow you to use the ```brew``` command
- Install qemu

```text
brew install qemu
```

You will have to exit your Terminal and start a new one for the PATH to be updated with the new qemu commands.

### Install Qemu on Ubuntu/Ubuntu-WSL

You may need to upgrade your Ubuntu installation first:

```text
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
```

After all this, then install Qemu:

```text
sudo apt install qemu qemu-system-misc
```

### Run and configure your local RISC-V image

Next, you need to download the Ubuntu RISC-V image I've created:

```text
mkdir cs631
cd cs631
curl https://www.cs.usfca.edu/~benson/cs315/files/ubuntu-22.04-riscv.zip --output ubuntu-22.04-riscv.zip
unzip ubuntu-22.04-riscv.zip
```

No you can cd into the vm directory and run the vm:

```text
cd ubuntu-22.04-riscv
./start.sh
```

It will take some time to boot up. Once you see the login prompt you can login using the ```ubuntu``` user:
- username: ubuntu
- password: goldengate

Alternatively, you can start a new terminal and ssh into the vm:

```text
ssh -p 4444 ubuntu@localhost
```

Use the same password above.

Now, you can just use the ubuntu user if you want or you can create a new user with your own username:

```text
sudo adduser <your_username>
```

This will ask you for a password and your full name. You can leave the rest of the fields blank.

If you do this and also want to have sudo (admin) access, which will be need to poweroff the vm, you need to do the following:

```text
sudo adduser <your_username> sudo
```

If you use the ubuntu user or if you created your own user, you can follow the ssh config instructions above to add your public key to the vm, then update your ssh config on your computer with a new entry for the local vm. For example here is what I use:

```text
Host riscv
  HostName 127.0.0.1
  Port 4444
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs631_2024s
  User benson
```

Note that before you copy your public key into the vm, you will want to ssh in and create the .ssh directory:

```text
ssh -p 4444 ubuntu@localhost (or ssh -p 4444 usernmae@localhost)
mkdir .ssh
chmod 700 .ssh
```

Then exit back to your computer and use scp to copy your public key into the vm like this:

```text
cd .ssh
scp -P 4444 .ssh/id_ed25519_cs631_2024s.pub <your_username>@localhost:.ssh
```

Go back into the vm and update ```authorized_keys``` and check the file permissions in .ssh.

Now you should be able to get into your local vm like this:

```text
ssh riscv
```

## Shutting down the local RISC-V vm

When you are done working in your local vm you should shut it down cleanly:

```text
sudo poweroff
```

## Setup ssh GitHub access

On your computer, you can add the following to ```~/.ssh/config```:

```text
Host github.com
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs631_2024s
  User <your_username>
```

After you do this, on the GitHub website go to the following page:
- [https://github.com/settings/keys](https://github.com/settings/keys)

Add your public key to GitHub

After you do this you can test your GitHub access on your computer like this:

```text
ssh -T git@github.com
Hi gdbenson! You've successfully authenticated, but GitHub does not provide shell access.
```

## Learn a Console-based editor: micro

The micro editor is a small but powerful console-based editor with intuitive key commands. We have micro installed on eurayle and Ubuntu RISV-V vms. To edit files, just type:

```text
$ micro <filename>
```

Here is a quick summary of commands:
```text
CTRL-Q - quit
CTRL-S - save
Shift-Arrow - select text
CTRL-C - copy selection
CTRL-X - cut selection
CTRL-V - paste selection
CTRL-F - find
```

You can learn more about micro and how to configure it [here](https://micro-editor.github.io).
