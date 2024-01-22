---
layout: default
title: Shell Usage
nav_order: 2
parent: Guides
permalink: /guides/shell-usage
---

# Shell Usage

We will be working extensively from the shell (commmand line) in this class. As such it is important that you become both familiar and comfortable working from the command line. Here are some basic concepts:

- You access the command line through a terminal program such as Terminal or iTerm on macOS and WSL Ubuntu Linux or Gitbash (installed from GitHub) on Windows.
- The shell prompt usually ends with a ```$``` (```sh``` or ```bash```) or ```%``` (```zsh```). After the prompt is where you can type your commands. Here are two prompts, the first from zsh on macOS and the second from bash on Linux:
  - ```benson@m2a ~ %```
  - ```[benson@stargate ~]$```
- After the prompt you can type a command.
- The commands are either built-in to the shell, such as ```cd```, or come from the file system such as ```ls```.
- The ```ls``` command is used to list the contents of a diretory:
  - ```ls``` with no arguments lists  the contents of the current direcory
  - ```ls <path>``` will list the contens of the directory specified by <path>
  - ```ls -al``` will show a detailed listing of files and all files, including those that start with a period ".". This is useful for seeing file permissions, size, and modification time.
- Commands that exist in the file system are found using the ```PATH``` environment variable.
- You can see the value of ```PATH``` like this:
  ```text
  [benson@stargate ~]$ echo $PATH
  /home/benson/.local/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/var/lib/snapd/snap/bin:/opt/riscv/bin
  ```
- When looking for a command in the ```PATH```, it is searched in order from the first directory to the last directory.
- You can change what is in the ```PATH``` in your ```.bash_profile``` (or ```.zprofile```) in your home directory.
- For example, you can add the following to ```.bash_profile``` or ```.zprofile```, to put ```~/.local/bin``` in your home directory as the first directory in the PATH:
  ```text
  export PATH=~/.local/bin:$PATH
  ```
- When you open a terminal window, your default directory (the current/present working directory) is your home directory, which is usually something like ```/Users/benson``` or ```/home/benson```)
- You can show your current directory with the pwd command like this:
  ```text
  $ pwd
  /Users/benson
  ```
- The ```~``` in a path is expanded to your home directory (e.g., /home/benson).
- To change to a directory use the ```cd``` command:
  - ```cd``` with no arguments returns you to your home directory
  - ```cd <path>``` takes you to <path>, were <path> can be absolute, like ```cd /home/benson```, or relative, like ```cd project01```
  - ```cd ..``` takes you up one directory level
- Use ```mkdir <name>``` to create a directory
- Use ```cp <source> <target>``` to copy a file, where <source> and <target> can be absolute and relative paths.
- Use ```mv <old_name> <new_name>``` to move (rename) a file from <old_name> to <new_name>
- To view the contents of a file, you can use:
  - ```cat <filename>```
- To view the contents of a large file, use ```less```, which allows you to "page" through a file with <space> to go forward a page and "b" to go back a page. You can also use the up/down arrows to scroll through a file one line at a time:
  - ```less <filename>``` 
- The ```cat``` command can also be used to creat a file or append to an existing file.
- To create a file with ```cat```, you can do the following:
  ```text
  $ cat > foo.txt
  FOO
  BAR
  ^D
  ```
  - After you type ```cat > foo.txt``` then press ENTER/RETURN, you can type or paste in the contents of the file. To end the file, press CTRL-D (^D), which means to press the CTRL key at the same time as the "D" key.
 - You can append to an existing file with ```cat``` like this:
   ```text
   $ cat foo.txt >> bar.txt
   ```
   - This will add the context of ```foo.txt``` to the end of ```bar.txt```
 - To save on typing on the command line you can create shell aliases
   - For bash, put aliases in ```~/.bash_aliases```
   - For zsh, put aliases in ```~/.zshrc```
 - Here are some example aliases
   ```text
   $ cat .bash_aliases
   alias lsa='ls -al'
   alias m='micro'
   alias p3='python3'
   alias sg='ssh stargate'
   alias ey='ssh euryale'
   alias eyvm='ssh euryalevm'
   ```
   - You will need to restart your terminal to have the changes take effect.
