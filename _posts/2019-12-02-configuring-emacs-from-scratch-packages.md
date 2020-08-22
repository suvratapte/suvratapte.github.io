---
layout: post
title:  Configuring Emacs from Scratch  -  Packages
author: Suvrat Apte
date:   2019-12-02 16:11:08 +0530
categories: emacs
comments: true
---

*(You can read this post on <a href="https://medium.com/@suvratapte/configuring-emacs-from-scratch-packages-220bbc5e55b7" target="_blank">Medium</a>.)*

This is the second part of a series on “Configuring Emacs from Scratch”.
You can read the first part <a href="/configuring-emacs-from-scratch-intro/" target="_blank">here</a>.

In the last part, we customized some defaults of Emacs. But Emacs is not at all limited to customizing the defaults. Emacs has a **huge** eco-system of external packages that you can install. The default package manager for Emacs is called “package”. Package can fetch packages from multiple sources. Elpa (Emacs Lisp Package Archive) is the source that it uses by default. But users usually add [Melpa](https://melpa.org/#/) and [Marmalade](https://marmalade-repo.org/) to their list of package sources.

In this part, we will add a few useful packages and learn how to configure them according to our needs.

## Installing packages

First, we will tell `package` to include Melpa in its list of package archives. To do that, add these lines to your `init.el`:

{% highlight elisp %}
(require 'package)
(package-initialize)
(add-to-list 'package-archives
             '("melpa" . "http://melpa.milkbox.net/packages/") t)
{% endhighlight %}

<!---excerpt-break-->

The first line tells Emacs to load the feature/file called “package”. This will load functions and variables from `package`.
The second line is some initialization that `package` does.
The third line tells package to add melpa to `package-archives`. In Elisp, a pair can be constructed with `(<element 1> . <element 2>)`. The last argument `t` means that the pair must be appended to the list. Without `t`, the pair would be prepended to `package-archives`.

Go to the end of `(require 'package)` line and execute it by pressing `C-x C-e`. Do the same with `(package-initialize)`. Now check what is the value of package-archives with `C-h v package-archives` as we have done all along. Execute the `add-to-list` function call and then check the value of `package-archives` to confirm that melpa is added. Also, notice that the value is a list of pairs.

Let’s see all the packages available for Emacs (from sources listed in package-archives of course). Press `M-x` and run `package-refresh-contents`. This will fetch updated content. Now you can see all the packages with `M-x package-list-packages`.

![package-list-packages](https://cdn-images-1.medium.com/max/3092/1*rwoP08Le2y9qmC84bZulnw.png)

*package-list-packages*

At the time of this writing, there are 4705 packages (from gnu + melpa). Now let’s install some package from this list. The default Emacs theme is quite unamusing, so let’s change the theme. Search for `spacemacs-theme` (at the time of writing this, it was on line #4008). Press `<RET>` while point is on the name. Details of `spacemacs-theme` will open up in a new window. Click on `install` (yes, I’m asking you to click using your mouse/trackpad. 😛) and then on Yes on the confirmation box.

This package contains a light and a dark version of the spacemacs theme, to apply the dark version, press `M-x load-theme <RET> spacemacs-dark <RET>`. Emacs will ask you if it is okay if this theme runs some code - answer with a `y`. Then it will ask if it should treat this theme as *safe* for future sessions - answer with a `y`. And there you have a beautiful dark theme (used by [Spacemacs](http://spacemacs.org/)) ! :)

Emacs must have added some code at the end of your `init.el`. When you said *yes* to mark this theme safe for future Emacs sessions, how is Emacs supposed to know this in future sessions? By adding code to `init.el`!
If you read the code, you will understand that it is just adding this theme in `custom-safe-themes`.

To tell Emacs to add such code to some other file instead of `init.el`, write this line in `init.el`:

{% highlight elisp %}
(setq custom-file "~/.emacs.d/custom-file.el")
{% endhighlight %}

Execute this line. Then create `custom-file.el` in `~/.emacs.d/` and paste the code added by Emacs to this file and remove it from `init.el`. Also, add this line in `init.el` to make sure that the `custom-file` is loaded.

{% highlight elisp %}
(load-file custom-file)
{% endhighlight %}

*Note: Adding `load-file` here might seem strange and unnecessary. But we will later see why it is required.*

Think about what we just did: We went to the list of all packages, installed a package, and then used it. What do you think will happen when you restart Emacs? Will the package be there? Will the theme stay?

Well, try it out! Quit Emacs (not with the cross button; with `C-x C-c`) and then start it again.

*What happened?*
Emacs went back to its old bland theme. :(

*Was the package there?
*It’s easy to check this. In your terminal, execute this command:

{% highlight shell %}
ls ~/.emacs.d/elpa/
{% endhighlight %}

You will see `spacemacs-theme` here. So the package *is* there on disk storage.
*(By default, Emacs stores it’s packages in the `elpa` directory.)*

*Why did this happen?*
It’s very simple. When you start Emacs, it opens `init.el` and executes the code in it. Is there any code in `init.el` which tells Emacs to use the `spacemacs-dark` theme? No!
So let’s add that code. At the end of `init.el`, add this line:

{% highlight elisp %}
(load-theme 'spacemacs-dark)
{% endhighlight %}

Execute this line and you’ll see that the theme is applied!
Now whenever you restart Emacs, it will start with the `spacemacs-dark` theme.

*Note: We had written a `load-file` line where I had said I will explain why it was required. Now is the time to explain that:
When we had loaded the theme for the first time, Emacs had asked us whether it was safe to use this theme in future sessions and then Emacs had added some code to mark it safe. We moved that code to `custom-file.el`. Now, we need to make sure that the code in `custom-file.el` is executed **before** the `load-theme` line, otherwise Emacs will again ask us the same questions. That is why the `load-file` line is necessary.*

As usual, do not forget to write good comments. Your `init.el` should look something like this:

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

;; Do not use `init.el` for `custom-*` code - use `custom-file.el`.
(setq custom-file "~/.emacs.d/custom-file.el")

;; Assuming that the code in custom-file is execute before the code
;; ahead of this line is not a safe assumption. So load this file
;; proactively.
(load-file custom-file)

;; Require and initialize `package`.
(require 'package)
(package-initialize)

;; Add `melpa` to `package-archives`.
(add-to-list 'package-archives
             '("melpa" . "http://melpa.milkbox.net/packages/") t)

;; Use the `spacemacs-dark` theme.
(load-theme 'spacemacs-dark)
{% endhighlight %}

## Customizing a package

Every package has a few variables through which you can customize its behavior. All the variables are prefixed by the package name. To see the variables of `spacemacs-theme`, press `C-h v spacemacs-theme- <TAB>`.

![](https://cdn-images-1.medium.com/max/3092/1*5ASSa1vNOPMTlZyXCNLpIA.png)

You can read details about each variable by pressing `C-h v <variable-name>`. I’m stressing on many `C-h` commands because it is very important to internalize the habit of using Emacs’ help system.

One thing I don’t like about the spacemacs theme is that the comments have a different background color. Let’s change that. You can do this by writing this code in `init.el`:

{% highlight elisp %}
(setq spacemacs-theme-comment-bg nil)
{% endhighlight %}

This line must be added before the `load-theme` line because the theme needs to be reloaded after setting this var.

Like this, to customize any package, you should look at all the variables that it exposes. It is very convinient to search variables exposed by a package since they are prefixed with their package name.

Another way to customize it to use the `customize` GUI menu. `customize` is extremely helpful and it is important to know this way of customization as well. So let’s make `spacemacs-theme` display comments in *italics* with `customize`.

Type `M-x customize <RET>`. You will see this:

![Customize menu](https://cdn-images-1.medium.com/max/3092/1*Vf8DFK8K7xFX8Nq_F3olDw.png)

*Customize menu*

Go to the search box and type `spacemacs-theme <RET>`. You will see this:

![spacemacs-theme customize menu](https://cdn-images-1.medium.com/max/3092/1*JVXBQbEFlO0Rcat2sEWbTA.png)

*spacemacs-theme customize menu*

The first toggle, `Spacemacs Theme Comment Bg` is off and is marked as `CHANGED outside Customize`. This is correct since we have changed it by writing Elisp in `init.el`.

Now expand `Spacemacs Theme Comment Italic` and click on `Toggle` to turn it on. Click on `Apply and Save` and click on `Yes` on the confirmation box. Press `q` to quit the `spacemacs-theme` customization buffer and then `q` again to quit the `customize` buffer (your point might be on the search box - in that case, move it out of the search box and then press `q`). Load the theme again for the change to take effect.

Now how will Emacs (or spacemacs-theme) know to make comments italic when it restarts? It must have added some elisp code somewhere. Guess where? The custom file!

Go to the customize file and you will see two changes - `spacemacs-theme-comment-bg` is being set to `nil` and `spacemacs-theme-comment-italic` is being set to `t`. These were the changed values (different from the default values) in the customize buffer. When you clicked on `Apply and Save`, Emacs added this code to `custom-file`.

Customize menu is a good place to discover customizations, but we should always bring the changes back to our Elisp code. So here, we will add the following line to `init.el` before the `load-theme` line:

{% highlight elisp %}
(setq spacemacs-theme-comment-italic t)
{% endhighlight %}

After this, we can remove these two lines form the custom file:
*(Do not unbalance paranthesis while doing this. Check paranthesis carefully.)*

{% highlight elisp %}
'(spacemacs-theme-comment-bg nil)
'(spacemacs-theme-comment-italic t)
{% endhighlight %}

One (annoying) thing you must have noticed is that we don’t have auto completion while writing Elisp. Let’s fix this.
Install the `company` (COMPlete ANY) package by typing `M-x package-install company <RET>`. To enable it for this and future Emacs sessions, write `(global-company-mode t)` in `init.el` and execute this.
Try writing some Elisp code, and you will see a small completion window displaying possible completions. For example, when you write `load-`, you will see this:

![Possible completions for “load-”](https://cdn-images-1.medium.com/max/2000/1*XHqOAbo_Nx8iIm4w32orlA.png)

*Possible completions for “load-”*

That’s nice! But how do you navigate between these completions?
Arrow keys. :(
We want to avoid moving away from the [standard touch typing position](https://en.wikipedia.org/wiki/Touch_typing#/media/File:QWERTY-home-keys-position.svg). So let’s change the key bindings and use `C-n` and `C-p` for navigating between the completions. Also, the completions appear after a delay of 0.5 seconds. Let’s make them appear without any delay.
For this, write this code at the end of your `init.el`:


{% highlight elisp %}
(define-key company-active-map (kbd "C-n") 'company-select-next)
(define-key company-active-map (kbd "C-p") 'company-select-previous)

(setq company-idle-delay 0.0)
{% endhighlight %}

Here, we are telling Emacs to use `C-n` for `company-select-next` and `C-p` for `company-select-previous`. We want these bindings only when we have auto completions buffer displayed. Otherwise, we will end up overriding the bindings to go to next and previous lines. So we tell Emacs to define these bindings only in `company-active-map`. There are package specific keymaps and it is very useful to define certain bindings only for a certain package.
Then we tell Emacs to show possible completions without any delay by setting `company-idle-delay` to `0.0`.

Execute these lines and try writing some more Elisp. It will much easier with auto-completion enabled. Try installing and customizing another package. Now you have powerful autocompletion in your arsenal! :)

As usual, do not forget to write good comments. Your `init.el` will look something like this:

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

;; Do not use `init.el` for `custom-*` code - use `custom-file.el`.
(setq custom-file "~/.emacs.d/custom-file.el")

;; Assuming that the code in custom-file is execute before the code
;; ahead of this line is not a safe assumption. So load this file
;; proactively.
(load-file custom-file)

;; Require and initialize `package`.
(require 'package)
(package-initialize)

;; Add `melpa` to `package-archives`.
(add-to-list 'package-archives
             '("melpa" . "http://melpa.milkbox.net/packages/") t)

;; Do not use a different background color for comments.
(setq spacemacs-theme-comment-bg nil)

;; Comments should appear in italics.
(setq spacemacs-theme-comment-italic t)

;; Use the `spacemacs-dark` theme.
(load-theme 'spacemacs-dark)

;; Use company mode everywhere.
(global-company-mode t)

;; Navigate in completion minibuffer with `C-n` and `C-p`.
(define-key company-active-map (kbd "C-n") 'company-select-next)
(define-key company-active-map (kbd "C-p") 'company-select-previous)

;; Provide instant autocompletion.
(setq company-idle-delay 0.0)
{% endhighlight %}

## Conclusion

In this part, we installed two packages and changed their default configuration. We saw how to discover configuration options by browsing variables or by going through the *Customize menu.* Customize menu is a great tool for discovering different configurations (but it is recommended to move the changed configurations back to your own Elisp code).

You would have also realized that the customization options in Emacs are far more granular than any of the IDEs. We can customize even the tiniest details and this gives us the power to make Emacs behave the way we want! :)

### Next: <a href="/configuring-emacs-from-scratch-use-package/">Making your setup organized and portable</a>
