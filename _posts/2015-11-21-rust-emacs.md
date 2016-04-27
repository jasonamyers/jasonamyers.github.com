---
layout: post
title: "Setting up Emacs for Rust Development"
description: "A deeper look at a rust development environment in Emacs"
tags: [emacs, rust, setup]
comments: true
share: true
---

When I code, I typically want the following from my editor:

*   Syntax highlight and indentation
*   Auto completion
*   Error checking
*   Jump to definition
*   Inline documentation

When coding in rust, I want these same features.  Thankfully there are a several great packages to help us achieve those desires. Before we do any futher I wanna make sure to give credit and a huge thanks to Bassam and his [blog post](https://bassam.co/emacs/2015/08/24/rust-with-emacs/). This is just to update based on what changes I found I needed.

### Installing Packages

You'll need to start by updating your package list contents by:

*   `M-x package-refresh-contents`

Next, we will install the follow packages in Emacs by M-x package-install <RET> <PackageName><RET>:

*   `company` - An autocompletion framework - [Website](http://company-mode.github.io/)
*   `company-racer` - A backend that enables company autocompletion with racer - [Website](https://github.com/emacs-pe/company-racer)
*   `racer` - A rust autocompletion engine - [Website](https://github.com/racer-rust/emacs-racer)
*   `flycheck` - on the fly syntax checking - [Website](https://github.com/flycheck/flycheck)
*   `flycheck-rust` - A rust backend for flycheck - [Website](https://github.com/flycheck/flycheck-rust)
*   `rust-mode` - An Emacs mode for editing rust files - [Website](https://github.com/rust-lang/rust-mode)

### Configuring Emacs

Now we need to configure these packages inside of our emacs configuration.  Here is a snippet of my .emacs.d/init.el:

<pre style="padding-left: 30px;">;; Reduce the time after which the company auto completion popup opens
(setq company-idle-delay 0.2)

;; Reduce the number of characters before company kicks in
(setq company-minimum-prefix-length 1)

;; Set path to racer binary
(setq racer-cmd "/usr/local/bin/racer")

;; Set path to rust src directory
(setq racer-rust-src-path "/Users/jasomyer/.rust/src/")

;; Load rust-mode when you open `.rs` files
(add-to-list 'auto-mode-alist '("\\.rs\\'" . rust-mode))

;; Setting up configurations when you load rust-mode
(add-hook 'racer-mode-hook #'company-mode)
(add-hook 'rust-mode-hook #'racer-mode)
(add-hook 'racer-mode-hook #'eldoc-mode)
(add-hook 'racer-mode-hook #'company-mode)
(global-set-key (kbd "TAB") #'company-indent-or-complete-common) ;
(setq company-tooltip-align-annotations t)</pre>

### Installing Rust

In case you don’t have Rust installed on your system you can install it either by:

*   Downloading the installation binary from [Rust’s website](https://www.rust-lang.org/install.html).
*   or using _Homebrew_: `brew install rust`.

### Building and Installing Racer

To generate the [Racer](https://github.com/phildawes/racer) binary that `company-racer` uses for it's magical powers, you will need to clone the _racer _repository from Github to your home directory and run `cargo build --release`.

*   `git clone https://github.com/phildawes/racer.git /tmp/racer`
*   `cd /tmp/racer`
*   `cargo build --release`
*   `mv /tmp/racer/target/release/racer /usr/local/bin`
*   `rm -rf /tmp/racer`

After running these commands and restarting your terminal you should be able to run the `racer` command which should complain about a missing `$RUST_SRC_PATH`.

### Setting $RUST_SRC_PATH

If you go back to the _elisp_ function you added to our `init.el` earlier you will be able to see that you defined `racer-rust-src-path` which points to `.rust/src` in your home directory. You will need to add the Rust source code there so Racer can use that to load methods for completion.

*   `git clone https://github.com/rust-lang/rust.git ~/.rust`
*   Add `export RUST_SRC_PATH=/Users/YOURUSERNAME/.rust/src` to your .bashrc or shell init file.
*   Restart your shell and emacs then open a Rust file.

You should now have all the features we desired earlier.  You can test this by:

1.  Open a rust file and try typing `use std::io::B` and press `<tab>. You should see some completion options.`
2.  Place your cursor over a symbol and hit `M-.` to jump to the definition.

If that does not work, make sure that racer works from the command line by doing the following:

*   <pre>racer complete std::io::B</pre>

You should see some autocompletion options show up.
