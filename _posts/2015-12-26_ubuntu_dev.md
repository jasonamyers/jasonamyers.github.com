---
layout: post
title: "Setting up Ubuntu Development Environments"
description: "How I like to setup my dev environment"
tags: [ubuntu, setup]
comments: true
share: true
---

1. Install Ubuntu (Whatever flavor you like) 
2. sudo apt-get upgrade && sudo apt-get upgrade 
3. Copy over SSH keys 
4. chmod 600 .ssh/* 
5. sudo apt-get install -y emacs git vim make gnome-tweak-tool curl build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget llvm libncurses5-dev 
6. Run Tweak Tool -> Click Typing -> set Caps Lock key behavior to "Make Caps Lock an additional Ctrl" -> set Alt/Win key behavior to "Alt and Meta are on Alt keys" 
7. Setup Firefox Sync 
8. Copy down dotfiles: git clone git@github.com:jasonamyers/dots 
9. cd dots 
10. ./setup.sh 
11. Download the lastest version of Go from https://golang.org/dl/ 
12. Install Go: sudo tar -C /usr/local -xzf go1.5.2.linux-amd64.tar.gz 
13. ./stage2-setup.sh 
14Open emacs, it will compile for a while and end in an error about go auto loads 
15. Run M-x update-file-autoloads, Use "~/Misc/emacs/go-mode.el/go-mode.el" as the first answer, and "~/Misc/emacs/go-mode.el/go-mode-load.el" as the autoloads files 
16. Exit and Save emacs 
17. Test it with: racer complete std::io::B 

Additionally

*   Install Slack https://slack.com/downloads
*   Install Virtualbox https://www.virtualbox.org/wiki/Linux_Downloads
*   Install Vagrant https://www.virtualbox.org/wiki/Linux_Downloads
