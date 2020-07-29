---
layout: post
title:  Configuring Emacs from Scratch  -  Intro
author: Suvrat Apte
date:   2019-11-12 18:13:05 +0530
categories: emacs
---

*(You can read this post on <a href="https://medium.com/@suvratapte/configuring-emacs-from-scratch-intro-3157bed9d040" target="_blank">Medium</a>.)*

## Intro

I have been using Emacs for 6 years now and for the last couple of years, Emacs has been my primary tool for writing Clojure at Helpshift Inc. Over the years, I have realized the value of Emacs. It has made me productive at work and org-mode has certainly made me more disciplined. The idea of having an editor which you can customize as it runs is itself amazing! :)

Unfortunately, I have also realized that getting on-boarded on Emacs is not an easy path. Many people try it for a few days and go back to their IDEs. So I’m going to try and write about customizing Emacs from scratch. My goal is to make this process simpler (or at least well documented) and help people get comfortable with Emacs until the point they realize its true value! :)

## Installation and interactive tutorial

I’m going to use Emacs 26.1 on macOS. You can get it here for [macOS](https://emacsformacosx.com/), [Linux](http://ubuntuhandbook.org/index.php/2019/02/install-gnu-emacs-26-1-ubuntu-18-04-16-04-18-10/) or for [Windows](https://www.thecoderworld.com/ide/how-to-install-emacs-text-editor-on-windows-10/).
*(The version doesn’t really matter. Most of the things written here will work for any Emacs version.)*

<!---excerpt-break-->

Once you start Emacs, you will see this:

![Startup screen](https://cdn-images-1.medium.com/max/3808/1*fv-ceMnQ6AHnhH0HWK3SRg.png)*Startup screen*

I would advise everyone to go through the “Emacs Tutorial” section displayed above. The Emacs way of doing things is a bit different and this interactive tutorial does a great job explaining it. It is a bit long but you won’t regret spending time on it.

But before you go through the tutorial, please consider mapping the Caps Lock key to control. Control is used a lot in Emacs, and if you don’t use Caps Lock as control, you’ll suffer from [*Emacs pinky*](http://onlineslangdictionary.com/meaning-definition-of/emacs-pinky). Here’s how to do it for [macOS](http://xahlee.info/kbd/osx_swapping_modifier_keys.html), [Linux](https://unix.stackexchange.com/questions/114022/map-caps-lock-to-control-in-linux-mint), and [Windows](https://superuser.com/questions/949385/map-capslock-to-control-in-windows-10).

If you don’t want to read this (you really should though!), here’s the gist of the tutorial:

### Tutorial Summary

*(If you have followed the interactive tutorial, skip this section and go to the **Customizing defaults** section)*

* Emacs uses the control and meta (`alt`/`option`) keys a lot. The key bindings with control and meta will be described as `C-<key>` and `M-<key>` respectively.

* Press `C-x C-c` (`C-x` and then `C-c`) to exit Emacs.

* `C-g` will quit the current operation. So if you are stuck and you don’t know what do to, press `C-g`.

* The cursor location is called “point” in Emacs.

* Use `C-f` for moving point forward, `C-b` for backward, `C-n` for going to next line and `C-p` for previous line.

* `C-a` for going to beginning of current line. `C-e` for end of current line.

* `C-v` for going to next page. `M-v` for previous page.

* The above key bindings when used with meta will work on a word level abstractions. For example, `M-f` will move the cursor one word ahead (vs one character with `C-f`). `M-a`, `M-e` will move between sentences.

* Files are opened in “buffers” in Emacs. There could be a buffer which is not associated with any file. But a file will always have a buffer associated with it when you open it in Emacs.

* `M-<` (Meta + Less than) will take you to beginning of a buffer and `M->` (Meta + Greater than) will take you to end of a buffer.

* Press `C-x k <RET>` to kill a buffer.

* Press `C-x 3` to split the Emacs window horizontally. Press `C-x 1` to make the split screen go away.

* Press `C-x C-f` to open a file.

* `C-x C-s` to save your edits.

* When you open a few file, the previous buffer still remains active. To go to the previous buffer, press `C-x b` and type the buffer name (file name) and press `<RET>`.

* To see the list of buffers, press `C-x C-b`. Press q to quit this window.

* The term “Windows” in Emacs is not the same as the one used by your OS. In Emacs, the OS “windows” are called “frames”. The split screens in Emacs are called “windows”. For example, when you split your screen, you can say that you have 2 “windows”.

* Use `C-x o` to move among windows.

* `C-d` will delete a character in backward direction.

* `C-k` will delete text from point to end of current line.

* To search for text, press `C-s` and type the text to be searched for. Pressing `C-s` after that will take you to the next occurence of the text. `C-s` moves in the forward direction and `C-r` moves in the backward direction. Both have a wrapping behavior. Press `RET` to stop search and move the point to the current occurence.

* If your point is on some text you want to search for, press `C-s` w to fill the search minibuffer with text under point.

* You can use prefix arguments to repeat commands. Press `C-u` to give a prefix argument to a command. For example, to go 10 pages ahead, you can press `C-u 10 C-v`. Or to add a line of 72 *s, press `C-u 70 *`.

* Instead of dragging to select text, press `C-<SPC>` (Control + Space) where you want to start dragging and then move the point to where you want to end. Text you have selected will be marked with a highlight. Press `C-w` to cut or `M-w` to copy.

* To paste, press `C-y`. Pasting is called “yanking” in Emacs terms. “Cutting” is called “killing”.

* You can also yank text that was killed previously, press `C-y` and then keep pressing `M-y`. You will go on seeing your kill history. This is because Emacs saves the text you kill in a ring data structure called “kill ring”.

* Press `C-/` to undo. Undo in Emacs is very different from other editors. *(You can read about it in the Tutorial; or just play with it to know how it works.)*

* **Emacs has a concept of modes. For any buffer, there is one major mode and there could be multiple minor modes. For example, if you open a python source code file, the major will be python-mode. Emacs will try to detect and use a proper major mode for any file that you open. Minor modes are modes that do small tasks such as auto indenting, spell checking, showing line numbers, etc. So more than one minor mode can be enabled per buffer.**

* The line that you see in the lower part of Emacs:

![Mode line](https://cdn-images-1.medium.com/max/2168/1*DdFCF87vYrfgkomIZ_7sAg.png)

*Mode line*

* It is called “mode line” and it displays useful information about the current buffer and the status of Emacs. For example, it displays which is the major mode that is currently enabled. In the above image, it is “Fundamental” mode.

## Customizing defaults

Now that we have completed the interactive Emacs tutorial, we don’t want Emacs to show the tutorial message when Emacs starts. When Emacs starts up, it looks for one of these 2 files: `~/.emacs.d/init.el` or `~/.emacs`. If it finds one of these, it will start executing the code in these files. The code in these files is supposed to be Elisp code. Emacs Lisp or Elisp is the language created for and used by Emacs.

So now we will go ahead and create this file.

* Press `C-x f` and write `.emacs.d/init.el` (`~/` will already be there) and press `<RET>`. A new buffer will open up.

* Write these contents in that buffer: (setq inhibit-startup-message t) and save (`C-x C-s`).

* Quit Emacs and restart.

* Emacs will not show the tutorial screen and instead it will start with a buffer named *scratch*. By convention, buffers which have names starting and ending with a * do not have any files associated with them.

Congratulations on your first Emacs customization! :)

Now let’s try to read and understand the Elisp code that we just wrote.

{% highlight elisp %}
(setq inhibit-startup-message t)
{% endhighlight %}

In simple terms, `setq` is a function which sets value of the given variable to the given value. In this case, it will set inhibit-startup-message to t.

Now is a good time to use the self documenting feature of Emacs. `setq` is a function and we can get help on functions by pressing `C-h f` (`C-h` for help and `f` for functions). Press `C-h f`, type `setq` and press `<RET>`. A window will open up a buffer named `*Help*` (again, not associated with a file) which will contain information about the setq function. Press `C-x 1` to maximize `*scratch*` again.

Now let’s see what `inhibit-startup-message` is. Since it is a variable, we will ask Emacs to tell us more about variables by pressing `C-h v`. Press `C-h v`, type `inhibit-startup-message` and press `<RET>`. Notice that for variables, their current value is also shown. This time to quit the documentation window, switch to it using `C-x o` and then press `q`.

Why was this help window quit when we pressed `q`? The `*scratch*` window will not quit on pressing `q`, it will just insert the letter q. Let’s find out why!

In Emacs, each key is bound to a function and there is a way to figure out which key is bound to which function. And these bindings are per buffer. So q must be bound to two different things as we see two different behaviors on pressing q.

Switch to the help buffer by pressing `C-x b *Help* <RET>`. Now let’s find out what `q` is bound to. Press `C-h k` and press `q`. “q runs the command quit-window”. Now we know why `q` is making the window go away; it was executing the function quit-window.

Let’s figure out what `q` is bound to in the `*scratch*` buffer. Quit the help window by pressing, guess what, `q`! You’ll be back on the `*scratch*` buffer. Now press `C-h k` and then `q`. “q runs the command self-insert-command”. `self-insert-command` just inserts the character that you have typed. So all the printable characters on the keyboard must be bound to `self-insert-command`. You can try verifying this. Also, you can read documentation for `self-insert-command` just like the way we read the documentation for `setq`.

Now we fully understand why `q` was acting differently in different buffers. And guess what, **we did not leave Emacs even once!** :)

*(Something fun to try on your own: Find out what C-h k is bound to.)*

Since we like to use the keyboard for doing all our tasks, the tool bar is not very useful for us.

![Tool bar](https://cdn-images-1.medium.com/max/2000/1*n1Odj3hdsp7B8tVfYf9WVg.png)


*Tool bar*

So let’s disable it. Go to your `init.el` buffer and write `(tool-bar-mode -1)` on a new line. Now let’s make this code take effect without restarting Emacs.

![init.el](https://cdn-images-1.medium.com/max/2000/1*nC9lUKQab7dYQB1x2pYzJg.png)

*init.el*

Execute this code by going one character after the closing parenthesis. Press `C-x C-e` to execute this code. The tool bar is gone!
Congratulations on your first live Emacs customization! :)

*(Now you should find out what `C-x C-e` is bound to. It is a good habit to always check which functions you are using instead of just remembering key bindings.)*

Now let us disable the menu bar and the scroll bar as well. Put this code in a new line in `init.el`:

{% highlight elisp %}
(menu-bar-mode -1)
(scroll-bar-mode -1)
{% endhighlight %}

To clearly see which line you are on, you can have current line highlighted with this code: `(global-hl-line-mode t)`. You will need to execute this line as we did before.

![global-hl-line-mode](https://cdn-images-1.medium.com/max/2000/1*dxvRS53yWa8KZco1VQNBMA.png)

*global-hl-line-mode*

Another important thing which is missing is knowing the line numbers and column numbers. For that, add `(line-number-mode t)`. You’ll see (line #, column #) coordinates of the point location in the mode line. To go to a particular line, press `M-g M-g`, enter the line number and press `<RET>`.

*If you use macOS, it is easier to use the “command” keys as “meta”. Add this line to make this happen: `(setq mac-command-modifier 'meta)`.*

What we have in the last 4 lines of Elisp above is just changing minor modes. We have disabled minor modes which display the tool bar, the menu bar and the scroll bar; we have enabled a minor mode which highlights the current line in the buffer. If you want more information on the currently enabled major and minor modes, press `C-h m`. On pressing this, a minibuffer will open up. Amogst other things, this buffer will describe commonly required key bindings for the respective modes.

Now that we have seen how to configure Emacs defaults to suit your needs, you can read the [Use better defaults](https://github.com/suvratapte/dot-emacs-dot-d/blob/e7f44d981004b44bb9ae23b6b67c421404ea6b4e/init.el#L19) section of my Emacs config and take the things that would be useful to you.
*[Linux and macOS users: Please go through the [Better interaction with X clipboard](https://github.com/suvratapte/dot-emacs-dot-d/blob/e7f44d981004b44bb9ae23b6b67c421404ea6b4e/init.el#L94).]*

Finally, let’s have a habit of writing comments for all the code that we have written. Our init.el will look like this:

{% highlight elisp %}
;; Do not show the startup screen.
(setq inhibit-startup-message t)

;; Disable tool bar, menu bar, scroll bar.
(tool-bar-mode -1)
(menu-bar-mode -1)
(scroll-bar-mode -1)

;; Highlight current line.
(global-hl-line-mode t)

;; Use `command` as `meta` in macOS.
(setq mac-command-modifier 'meta)
{% endhighlight %}

## Conclusion

Throughout this session, we have used `C-h <some key>` many times. `C-h` is the prefix key for getting help in Emacs. We have used it quite well here and you should continue following this pattern of using help to know more about Emacs! :)

Knowing how to use the help system is crucial for mastering Emacs!

### Next: <a href="/emacs/2019/12/02/configuring-emacs-from-scratch-packages.html">Installing and configuring new packages</a>
