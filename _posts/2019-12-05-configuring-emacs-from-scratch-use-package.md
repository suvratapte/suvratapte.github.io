---
layout: post
title:  Configuring Emacs from Scratch  -  use-package
author: Suvrat Apte
date:   2019-12-05 22:05:37 +0530
categories: emacs
comments: true
---

*(You can read this post on <a href="https://medium.com/@suvratapte/configuring-emacs-from-scratch-use-package-c30382297877" target="_blank">Medium</a>.)*

This is the third part of a series on “Configuring Emacs from Scratch”.
You can read the first part <a href="/configuring-emacs-from-scratch-intro/" target="_blank">here</a> and the second part <a href="/configuring-emacs-from-scratch-packages/" target="_blank">here</a>.

In the last two parts, we customized some defaults of Emacs. We also installed two packages and customized their default behavior. In this part, we will organize our configuration and make it portable using ***use-package***.

<!---excerpt-break-->

After the first two parts, our ``init.el`` looks like this:

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

;; Do not use ``init.el`` for `custom-*` code - use `custom-file.el`.
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

What problems do you see in this configuration? There is one major and one minor problem *(and there must be many which I have missed)*.

1. This setup is not portable.

1. The structure of code is too flat.

### This setup is not portable

We cannot just take our `init.el` from one computer to other and then expect to have our Emacs setup portes. What do you think will happen when you copy your `init.el` to a new computer?

*What is our code assuming?
*Is our code telling Emacs to install `spacemacs-theme` and `company`? No!
We are assuming that the packages - `spacemacs-theme` and `company` will already be there on all computers.

When you open Emacs on a new computer, it will only have the default Emacs packages. So the *load-theme* call in our `init.el` will fail. Enabling `company-mode` will also fail since there is nothing called `company-mode` at this point. So all your configuration code related to `spacemacs-theme` and `company` will either fail or will be ineffective.
An example where it will not fail but will be ineffective is when you set `spacemacs-theme-comment-bg` to `nil`. This call will not fail. It will just create a variable. But it will be ineffective since there is no package which uses that variable.

So the conclusion is - **There is no way to port this setup without manually installing all the packages.**

### The structure of the code is too flat

Currently, our configuration code is not grouped based on packages. You can always group it manually by keeping all configurations for a package together. But that can get hard to maintain. As you go on adding packages and customizing little things here and there, it is easy to spill configuration of a package all over `init.el`.

This might not even feel like a problem at this point in time. But as our setup evolves, this will be a big problem for managing the setup.

There are many different solutions for the above two problems. The natural way of solving the first problem will be to maintain a list of packages that we use and check if those are installed. If some package is not installed, install it. Something like this:

{% highlight elips %}
(defvar my-packages '(spacemacs-theme company))

(dolist (p my-packages)
  (when (not (package-installed-p p))
    (package-install p)))
{% endhighlight %}

*Note: In Elips any function ending with a `-p` is a predicate.*

This solution works perfectly well.
But there is a very elegant solution for this (which happens to solve the second problem as well).
It is called ***use-package***.

## Use-package

***[use-package](https://github.com/jwiegley/use-package)*** is an Elisp [macro](https://www.gnu.org/software/emacs/manual/html_node/elisp/Macros.html) written by **[John Wiegley](https://github.com/jwiegley)**. It simplifies and groups together configuration for packages. Install use-package by pressing:
*(By now, you should know how to install a package.)*

{% highlight elisp %}
M-x package-install <RET> use-package <RET>
{% endhighlight %}

A common use-package declaration looks like this:

{% highlight elisp %}
(use-package <package-name>
  :init
  <code to be executed before loading the package>

  :config
  <code to be executed after loading the package>

  :bind
  <key bindings for this package>)
{% endhighlight A%}

*Note: Any word, preceded with a `:` is called a `keyword` in Elisp. You can consider keywords as being similar to strings, but with a different purpose and presence.*

Let’s see how would a use-package declaration look for `company`. Our current config for `company` is:

{% highlight elisp %}
(global-company-mode t)

(define-key company-active-map (kbd "C-n") 'company-select-next)
(define-key company-active-map (kbd "C-p") 'company-select-previous)

(setq company-idle-delay 0.0)
{% endhighlight %}

With *use-package*, it will look like this:

{% highlight elisp %}
(use-package company
  :bind (:map company-active-map
         ("C-n" . company-select-next)
         ("C-p" . company-select-previous))
  :config
  (setq company-idle-delay 0.3)
  (global-company-mode t))
{% endhighlight %}

Here we are saying-

1. This is a use-package declaration for the package `company`.

1. In `company-active-map`, bind `C-n` to `company-select-next` and `C-p` to `company-select-previous`.
*Notice the minimal syntax here vs the `define-key` call.*

1. After `company` is loaded, set `company-idle-delay` to `0.3` and enable company mode everywhere.

Now all the configuration related to `company` is naturally grouped together in the `use-package` declaration. Next time when we want to make some changes to `company`, we will come to this declaration and add our code. This way, our setup will always be well grouped. So use-package has solved our second problem.

*But what about the first problem? Will use-package download the missing packages?*

Yes, it will. You just have to add `:ensure t` to the declaration. For example, look at this:

{% highlight elisp %}
(use-package magit
  :ensure t
  :bind ("C-x g" . magit-status))
{% endhighlight %}

*Note: [Magit](https://magit.vc/) is an super awesome git client for Emacs that you **must** use.*

`:ensure t` will make sure that `magit` is downloaded if it is not there. Also, look at the `bind` syntax here. When you want to make a global binding (unlike the `bind` in `company`, which was local to `company-mode-map`), the syntax is even more minimal. You just have to specify `(key-binding . command)` pairs. :)

***use-package*** has solved both of our problems! :)

*Note: use-package has many more useful (and very well thought of) keywords. I recommend going through the [README.md](https://github.com/jwiegley/use-package).*

*Note: Even though use-package is a handy macro, we must know what is going on under the hood when we use it. Since it is just a macro, you can expand it yourself and see what is going on. Read about macro expansion [here](https://www.gnu.org/software/emacs/manual/html_node/elisp/Expansion.html).*

*So is our setup fully portable now?
*Nope! There is still one small problem. use-package is not there in Emacs by default and you cannot use use-package to install itself. :P

So to make our setup fully portable, we need to check if use-package is installed and if it is not, we need to install it.

{% highlight elisp %}
(when (not (package-installed-p 'use-package))
  (package-refresh-contents)
  (package-install 'use-package))
{% endhighlight %}

Now our `init.el` will look something like this:

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

(when (not (package-installed-p 'use-package))
  (package-refresh-contents)
  (package-install 'use-package))


;; Additional packages and their configurations

(use-package spacemacs-theme
  :config
  ;; Do not use a different background color for comments.
  (setq spacemacs-theme-comment-bg nil)

  ;; Comments should appear in italics.
  (setq spacemacs-theme-comment-italic t)

  ;; Use the `spacemacs-dark` theme.
  (load-theme 'spacemacs-dark))


(use-package company
  ;; Navigate in completion minibuffer with `C-n` and `C-p`.
  :bind (:map company-active-map
         ("C-n" . company-select-next)
         ("C-p" . company-select-previous))
  :config
  ;; Provide instant autocompletion.
  (setq company-idle-delay 0.3)

  ;; Use company mode everywhere.
  (global-company-mode t))
{% endhighlight %}

Here are a few packages (along with their configurations) that I think every Emacs setup should have:

{% highlight elisp %}
;; Recent buffers in a new Emacs session
(use-package recentf
  :config
  (setq recentf-auto-cleanup 'never
        recentf-max-saved-items 1000
        recentf-save-file (concat user-emacs-directory ".recentf"))
  (recentf-mode t)
  :diminish nil)

;; Display possible completions at all places
(use-package ido-completing-read+
  :ensure t
  :config
  ;; This enables ido in all contexts where it could be useful, not just
  ;; for selecting buffer and file names
  (ido-mode t)
  (ido-everywhere t)
  ;; This allows partial matches, e.g. "uzh" will match "Ustad Zakir Hussain"
  (setq ido-enable-flex-matching t)
  (setq ido-use-filename-at-point nil)
  ;; Includes buffer names of recently opened files, even if they're not open now.
  (setq ido-use-virtual-buffers t)
  :diminish nil)

;; Enhance M-x to allow easier execution of commands
(use-package smex
  :ensure t
  ;; Using counsel-M-x for now. Remove this permanently if counsel-M-x works better.
  :disabled t
  :config
  (setq smex-save-file (concat user-emacs-directory ".smex-items"))
  (smex-initialize)
  :bind ("M-x" . smex))

;; Git integration for Emacs
(use-package magit
  :ensure t
  :bind ("C-x g" . magit-status))

;; Better handling of paranthesis when writing Lisps.
(use-package paredit
  :ensure t
  :init
  (add-hook 'clojure-mode-hook #'enable-paredit-mode)
  (add-hook 'cider-repl-mode-hook #'enable-paredit-mode)
  (add-hook 'emacs-lisp-mode-hook #'enable-paredit-mode)
  (add-hook 'eval-expression-minibuffer-setup-hook #'enable-paredit-mode)
  (add-hook 'ielm-mode-hook #'enable-paredit-mode)
  (add-hook 'lisp-mode-hook #'enable-paredit-mode)
  (add-hook 'lisp-interaction-mode-hook #'enable-paredit-mode)
  (add-hook 'scheme-mode-hook #'enable-paredit-mode)
  :config
  (show-paren-mode t)
  :bind (("M-[" . paredit-wrap-square)
         ("M-{" . paredit-wrap-curly))
  :diminish nil)
{% endhighlight %}

## Conclusion

In this three part series on configuring Emacs from scratch, we have seen,

* The Emacs way of text editing.

* Using the Help system to know what is going on.

* Tweaking the defaults of Emacs.

* Installing packages.

* Organizing your setup.

* Making your setup portable.

While doing this, we have seen how customizable Emacs is! :)

I have not covered even a percent of the capabalities that Emacs has. [Org-mode](https://orgmode.org/) in itself is a topic of another (huge) series of posts. And there are many more packages which make Emacs special and unique.

But I think we have covered enough to be able to proceed on our own now.
**The key is to know how to use Emacs’s help system.**
*(Sorry for repeating this over and over again, but you will realize the power and convenience of Emacs’s help as you use it.)*

You can find my Emacs setup [here](https://github.com/suvratapte/dot-emacs-dot-d). Feel free to borrow things. If you find better ways, please send pull requests. :)
*(It is strongly recommended to have your Emacs configuration under version control.)*

*I must mention and thank [Narendra Joshi](https://github.com/narendraj9) and [Vedang Manerikar](https://github.com/vedang) as I’ve always stolen (and look forward to stealing) things from their Emacs configurations.*

## Tips on “Evolving your Emacs setup”

The strategy I use to improve my Emacs setup is called **UFFL**.
*(Honestly, only I call it UFFL.)*

### **Use - Find painpoints - Fix - Loop**

*As you would have guessed, I have totally made this up just to make it analogous to [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop). But it does describe the strategy well. :)*

After making an edit to my configuration, I use it for a few days for my day to day tasks. I often find some painpoints *(i.e. things that can be improved / automated / achieved in lesser number of key strokes)* while using it. Then I find solutions to those problems.
Usually, I try to solve the problems myself by finding the right configuration parameters or by writing some Elisp. Then I check if there are solutions on [Emacs Stack Exchange](https://emacs.stackexchange.com/) (or anywhere on the Internet). In most of the cases, solutions from the Internet are better and more elegant than my own solutions.
But I still recommend finding/writing your own solution before looking up online, so that you get better at Elisp and at the Emacs way of thinking. :)

And this goes on forever …



## The joy is in the journey!

![](https://cdn-images-1.medium.com/max/2000/1*6O2fiWWJy7Ban_oF5AJejg.png)
