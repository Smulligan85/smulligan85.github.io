---
layout: post
title: "How to Learn Vim Without Going Insane Part 2"
date:   2016-11-22
categories: tutorial
---

This blog post is part 2 of continuing series on getting started with Vim. If you haven't read part 1 yet, you can read it [here](http://smulligan85.github.io/tutorial/2016/06/01/how-to-learn-vim-without-going-insane.html). Done? Great! Now that you've spent a few weeks familiarizing yourself with how to navigate and edit files the "Vim Way" let's cover how to get started actually using Vim. If you're on a Unix system you most likely already have Vim installed and can access it from the terminal with the command `vim`. If for some reason you don't have Vim already installed you can follow the download instructions for your operating system [here](http://www.vim.org/download.php). 

To users new to Vim the out of the box version is pretty basic. This is true, but also one of the biggest benefits of Vim is the ability to customize to each developers needs. This is where your `.vimrc` file comes in. If you've googled around customizing Vim before you've probably come across developers referring to their `.vimrc` file or dot files. This is the file or group of files that allows you to fully customize Vim. There are tons of examples of custom dot files available on Github. One of the earliest presentations I saw on getting started with Vim was this [talk](https://www.youtube.com/watch?v=_NUO4JEtkDw) by Mike Coutermarsh. After watching the presentation I immediately cloned Thoughtbot's dotfiles and dove into the world of Vim. However, looking back I regret cloning someone else's dotfiles. 

Firstly, as a new Vim user I was understandably ignorant to many of the customizations included in the dotfiles. Secondly, the Thoughtbot dotfiles are pretty extensive and included some files and packages that really weren't necessary. Instead I would recommend an alternative approach. Instead of cloning someone else's dotfiles I recommend using them as a guide for building out your own. [Thoughtbot's dotfiles](https://github.com/thoughtbot/dotfiles) are a great place to start. Specifically, review the `.vimrc` and `.vimrc.bundles` files. Let's cover a few sections of the `.vimrc` that I found helpful starting out.

The first line I recommend adding is remapping the `ESC` key to `jj` to more easily enter normal mode. To remap add the following line to your `.vimrc` file: `:imap jj <Esc>`. If you prefer you can remap to `jk`, but I prefer the double j. Another great addition is to add the following line to help cement that use of the hjkl keys.
```
nnoremap <Left> :echoe "Use h"<CR>
nnoremap <Right> :echoe "Use l"<CR>
nnoremap <Up> :echoe "Use k"<CR>
nnoremap <Down> :echoe "Use j"<CR>
```
This group of lines will display a warning any time you use the arrow keys instead of the hjkl keys. Positive reinforcement! A important setting to include is to set your leader key. To set it to a space add the following line: `let mapleader = " "`. I use the leader key as a way of remapping common commands. For example I have saving a file remap to `<Leader>s` or creating a new tab as `<Leader>t`. There are a ton of more options in the Thoughtbot dotfiles to check out. I also recommend googling Vim dotfiles to find other interesting configurations. 

The next post will cover adding a plugin manager and go over some universal plugins that are great for increasing workflow. Happy coding.
