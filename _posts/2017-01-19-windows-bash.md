---
layout: post
title: "Notes on Windows Bash"
date: '2017-01-19'
categories: productivity windows bash
author: brasskazoo
---

I've been using the Ubuntu flavour [Windows Linux Subsystem](https://docs.microsoft.com/en-us/windows/wsl/about) for over 6 months now, and its now an essential tool to my workflow. Coming from a Linux and Mac background, having a tool offering similar features is a massive bonus.

It's not without its problems though, and its neccesary to be aware of its limitations (especially when it comes to launch Windows executables vs linux executables).

Concepts
---

 * The linux home (`~/`) is not the same as Windows home. In fact the two user accounts are completely separate, with different passwords.
 * Windows C: can be found at `/mnt/c/` - and user directory is as `/mnt/c/Users/<username>`
 * Consider creating symlinks to important directories - e.g. `ln -s /mnt/c/Users/<user>/Documents ~/Documents` to create a link to your Windows Documents directory
 * Linux applications are installed as normal via `apt-get` - e.g. `git`, `python`, `nodejs` etc.
 

Managing Customisations
---
I've created a `~/.bash_aliases` and `~/.bash_custom` file to manage customisations to my WLS environment (and make it easy to migrate to another machine if neccesary). Both are referenced in `~/.bashrc` like this:

    # Alias definitions.
    
    if [ -f ~/.bash_aliases ]; then
        . ~/.bash_aliases
    fi

    ...
    
    # CUSTOMISATIONS 
    # Use a stand-alone file for environment customisations
    #
    if [ -f ~/.bash_custom ]; then
        . ~/.bash_custom
    fi

Launching Sublime Text via an alias (or Atom, or any other Windows program)
---

In `~/.bash_aliases`, I have created an alias to `subl.exe` which will open a new sublime window:

`alias sublime='/mnt/c/Program\ Files/Sublime\ Text\ 3/subl.exe -n '`

I use this in conjunction with a symlink to a Windows `Projects` directory, so I can pass in a directory or a filename that will be opened in Sublime Text:

    sublime .
    sublime README.md
    sublime some-project/

**Note:** This works well under symlinks to Windows directories - it doesn't work directly on WLS files and folders, e.g. I can't use this to open `~/.bashrc` or `~/`, I guess because the path isn't easily translated to the Windows equivalent due to how the WLS directories are organised 

Copying ssh keys from Windows
---

In `~/.bash_custom`, I use `rsync` to copy files from my Windows home to my linux `~/.ssh/` directory. This allows me to keep my linux environment in sync while using Windows applications that might require the SSH keys (e.g. git commits through an IDE).

    rsync -a --ignore-existing --chmod=600 --no-owner --no-group /mnt/c/Users/wdampney/.ssh/ ~/.ssh
    
**Note** Another caveat: if you edit the files that come from Windows in say, `vim`, something happens with permissions and `rsync` loses the ability to overwrite it - and so the attempt to copy the file over fails. If this happend, you need to tack down the file in Windows Explorer and delete it.

Using Vagrant
---
Vagrant is a large component of my development workflow, as a means to standardise linux-based tools across my team.

From https://www.vagrantup.com/docs/other/wsl.html:
> Vagrant must be installed within Ubuntu on Windows. Even though the vagrant.exe file can be executed from within the WSL, it will not function as expected.

The Windows Vagrant exe can't be launched from WLS - you'll need to install the deb package and set the following in your bash profile:

    export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
    
Once that is done, you can use the usual `vagrant up` and `vagrant ssh` commands as usual - and Vagrant will handle the mapping of functions and devices from the linux Vagrant (`/usr/bin/vagrant`) to Windows Vagrant and VirtualBox.

 Command Prompt Customisations
 ---
 
I highly recommend using Solarized colour schemes - https://github.com/neilpa/cmd-colors-solarized - and use it with Windows Bash by discovering the bash shortcut (In Start menu, seach for 'bash', right click and 'Open file location'), and running:
 
    Update-Link.cmd "<Shortcut Path>" dark
    
I also use a customised prompt to display git branches in `~/.bashrc` (this took a bit of trial-and-error, so YMMV):

    if [ "$color_prompt" = yes ]; then
        #PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
        PS1="${debian_chroot:+($debian_chroot)}\[\033[01;96m\]\u\[\033[38;5;7m\]@\[\033[38;5;94m\]\l \[$(tput sgr0)\]\[\033[38;5;6m\][\[\033[38;5;3m\]\W\[\033[38;5;6m\]]\[\033[38;5;9m\]\$(__git_ps1) \[$(tput sgr0)\]\[\033[38;5;7m\]\\$ \[$(tput sgr0)\]"
else
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
    fi
    
    
    
