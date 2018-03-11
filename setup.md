---
layout: page
title: Setup
root: .
---

# Programs

In this part of the workshop we'll be using Unix and Git and will also need an account on GitHub. 
If you are using MacOS or Linux computer, both UNIX and Git are already available to you.

## Unix
### Linux
On most versions of Linux, Unix is accessible by running the Terminal program, which can be found 
via the applications menu or the search bar. 

### Mac OS
For a Mac computer, Unix can be accessed via the Terminal Utilities program within your Applications folder.

### Windows
Computers with Windows operating systems do not automatically have a Unix Terminal program installed.
A Unix environment can be emulated using the “Git Bash” program, by partitioning your hard drive 
and installing Ubuntu (a free version of Linux), or by installing Cygwin. In this lesson, we recommend 
you to use an emulator included in Git for Windows, which gives you access to both Bash shell commands 
and Git.

* Git Bash: [https://openhatch.org/missions/windows-setup/install-git-bash](https://openhatch.org/missions/windows-setup/install-git-bash) 
* Ubuntu: [https://help.ubuntu.com/community/Wubi](https://help.ubuntu.com/community/Wubi)
* Cygwin: [https://www.cygwin.com](https://www.cygwin.com)

Alternatively,  
* you can use an [online emulator](https://www.technonutty.com/2015/07/9-online-linux-terminal-emulator-for-practice.html) or
* you can use a Bash shell command-line tool [available for Windows 10](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/).

# Setting up Git

When we use Git on a new computer for the first time,
we need to configure a few things.On a command line, Git commands are written as `git verb`,
where `verb` is what we actually want to do. We use `config` verb for configurations:

~~~
git config --global user.name "Your Name"
git config --global user.email "your@email.xxx"
git config --global color.ui "auto"
~~~
{: .bash, eval=FALSE}

Please use your own name and email address instead of Dracula's. 
This user name and email will be associated with your subsequent Git activity,
which means that any changes pushed to
[GitHub](https://github.com/),
[BitBucket](https://bitbucket.org/),
[GitLab](https://gitlab.com/) or
another Git host server in a later lesson will include this information.

> ## Line Endings
>
> As with other keys, when you hit the 'return' key on your keyboard, 
> your computer encodes this input as a character. Different operating systems
> use different character(s) to represent the end of a line. 
> Because git uses these characters to compare files, 
> it may cause unexpected issues when editing a file on different machines. 
> 
> The following settings are recommended to avoid these problems:
>
> On OS X and Linux:
>
> ~~~
> $ git config --global core.autocrlf input
> ~~~
> {: .bash}
>
> And on Windows:
>
> ~~~
> $ git config --global core.autocrlf true
> ~~~
> {: .bash}
> 
> You can read more about this issue 
> [on this GitHub page](https://help.github.com/articles/dealing-with-line-endings/).
{: .callout}

For these lessons, we will be interacting with [GitHub](https://github.com/) and so the email 
address used should be the same as the one used when setting up your GitHub account. If you are 
concerned about privacy, please review [GitHub's instructions for keeping your email address private][git-privacy]. 

By default, git uses VI as the text editor.  You can set your favorite text editor, following this table:

| Editor             | Configuration command                            |
|:-------------------|:-------------------------------------------------|
| Atom | `$ git config --global core.editor "atom --wait"`|
| nano               | `$ git config --global core.editor "nano -w"`    |
| BBEdit (Mac, with command line tools) | `$ git config --global core.editor "bbedit -w"`    |
| Sublime Text (Mac) | `$ git config --global core.editor "subl -n -w"` |
| Sublime Text (Win, 32-bit install) | `$ git config --global core.editor "'c:/program files (x86)/sublime text 3/sublime_text.exe' -w"` |
| Sublime Text (Win, 64-bit install) | `$ git config --global core.editor "'c:/program files/sublime text 3/sublime_text.exe' -w"` |
| Notepad++ (Win, 32-bit install)    | `$ git config --global core.editor "'c:/program files (x86)/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"`|
| Notepad++ (Win, 64-bit install)    | `$ git config --global core.editor "'c:/program files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"`|
| Kate (Linux)       | `$ git config --global core.editor "kate"`       |
| Gedit (Linux)      | `$ git config --global core.editor "gedit --wait --new-window"`   |
| Scratch (Linux)       | `$ git config --global core.editor "scratch-text-editor"`  |
| emacs              | `$ git config --global core.editor "emacs"`   |
| vim                | `$ git config --global core.editor "vim"`   |

It is possible to reconfigure the text editor for Git whenever you want to change it.

The four commands we just ran above only need to be run once: the flag `--global` tells Git
to use the settings for every project, in your user account, on this computer.

You can check your settings at any time:

~~~
$ git config --list
~~~
{: .bash}

You can change your configuration as many times as you want: just use the
same commands to choose another editor or update your email address.

> ## Proxy
>
> In some networks you need to use a
> [proxy](https://en.wikipedia.org/wiki/Proxy_server). If this is the case, you
> may also need to tell Git about the proxy:
>
> ~~~
> $ git config --global http.proxy proxy-url
> $ git config --global https.proxy proxy-url
> ~~~
> {: .bash}
>
> To disable the proxy, use
>
> ~~~
> $ git config --global --unset http.proxy
> $ git config --global --unset https.proxy
> ~~~
> {: .bash}
{: .callout}

> ## Git Help and Manual
>
> Always remember that if you forget a `git` command, you can access the list of commands by using `-h` and access the Git manual by using `--help` :
>
> ~~~
> $ git config -h
> $ git config --help
> ~~~
> {: .bash}
{: .callout}

[git-privacy]: https://help.github.com/articles/keeping-your-email-address-private/


### Get a GitHub User Account

Create a user account on GitHub if you don't have one already. 
Be sure to choose a user ID that you are happy using for the rest 
of your professional career. GitHub is a very important tool for 
computational biology and if you continue working in data-intensive 
fields, you will be using this account again in the future. Join GitHub here:
[https://github.com/join](https://github.com/join).

We will use the SSH protocol to interact with GitHub so we need to create/use SSH keys.  

* Go to you home directory by entering the `cd` command in your terminal.
* Enter `ls -a` command in your home directory.
* If `.ssh` directory does not exist, issue `ssh-keygen -t rsa` command.
* Enter the `.ssh` with the `cd .ssh` command and print the content of the id_rsa.pub file 
  with `cat id_rsa.pub`.
* Copy the entire contents of this file and add it to the SSH and GPG keys section in your 
  GitHub profile Settings.
  
![](https://codenvy.com/docs/assets/imgs/Clipboard6.jpg)  

## Additional tutorials

[Git/GitHub](https://isu-molphyl.github.io/EEOB563-Spring2018/computer_labs/lab1/git.pdf)  
[VI](https://isu-molphyl.github.io/EEOB563-Spring2018/computer_labs/lab1/vi_tutorial.pdf)  


> #### References
> * [Git for Windows](https://git-for-windows.github.io/)
> * [How to Install Bash shell command-line tool on Windows 10](https://www.windowscentral.com/how-install-bash-shell-command-line-windows-10)
> * [Install and Use the Linux Bash Shell on Windows 10](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)
> * [Using the Windows 10 Bash Shell](https://www.howtogeek.com/265900/everything-you-can-do-with-windows-10s-new-bash-shell/)
> * [Using a UNIX/Linux emulator (Cygwin) or Secure Shell (SSH) client (Putty)](http://faculty.smu.edu/reynolds/unixtut/windows.html)
{: .callout}

### Windows Users





