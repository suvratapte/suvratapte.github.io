---
layout: post
title:  Org Capure Talk - Emacs Meetup
author: Suvrat Apte
date:   2021-04-02 13:13:40 +0530
categories: emacs, org-mode, org-capture, productivity
comments: true
---

This is a talk that I had given at an Emacs meetup hosted by Helpshift.

<iframe width="672" height="378" src="https://www.youtube.com/embed/tFt6plDQm58" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<br/>
<!---excerpt-break-->

Here is the code that was shown in this talk:

{% highlight elisp %}
;; This is the demo file that was used for this meetup: https://www.meetup.com/the-peg/events/270312246/

;;; Code:

;; ──────────────────────────── Org mode vars - Default values ──────────────────────────

(setq org-directory "~/org")
(setq org-capture-templates nil)


;; ─────────────────────────────────── Basic templates ──────────────────────────────────

(setq suv-org-personal-todo-file (concat org-directory "/todo.org"))

;; `org-capture-templates` should be a list of template specifications:
;; Each specification (<key> <short description> <type> <target> <template> <properties>)

(setq org-capture-templates
      '(("t"
         "Personal todo"
         entry
         (file suv-org-personal-todo-file)
         "* TODO %^{Description}")))

;; Settting cursor position.
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description}\n  %?")))

;; Adding date in logbook.
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description}\n  :LOGBOOK:\n  - Added: %U\n  :END:\n  %?")))


;; ──────────────────────────────── Advanced configuration ────────────────────────────────

;; Add tags.
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")))

;; Add template for meeting notes.
(setq suv-org-meeting-notes-file (concat org-directory "/meeting-notes.org"))

(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] ")))

;; Prepend
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] "
         :prepend t)))

;; Clock-in and clock-resume
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] "
         :prepend t
         :clock-in t
         :clock-resume t)))

;; Movies
(setq suv-org-movies-file (concat org-directory "/movies.org"))

;; Immediate-finish
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] "
         :prepend t)

        ("M" "Movie" entry (file suv-org-movies-file)
         "* TODO %^{Description}"
         :immediate-finish t)))

;; Work
(setq suv-org-work-file (concat org-directory "/work.org"))

;; Auto complete for variables and using the variables
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] "
         :prepend t)

        ("M" "Movie" entry (file suv-org-movies-file)
         "* TODO %^{Description}"
         :immediate-finish t)

        ("w" "Work task" entry (file suv-org-work-file)
         "* TODO %^{Type|TODO|DEP|BUG}-%^{Ticket number} - %^{Description}
  :PROPERTIES:
  :LINK:     https://helpshift.atlassian.net/browse/%\\1-%\\2
  :END:
  :LOGBOOK:\n  - Added - %U\n  :END:\n  ")))


;; ────────────────────────────── Writing code in templates ─────────────────────────────

(setq suv-org-reading-list-file (concat org-directory "/reading-list.org"))

;; Get from kill ring
(setq org-capture-templates
      '(("t" "Personal todo" entry (file suv-org-personal-todo-file)
         "* TODO %^{Description} %^g\n  :LOGBOOK:\n - Added: %U\n  :END:\n  %?")

        ("m" "Meeting notes" entry (file suv-org-meeting-notes-file)
         "* %^{Agenda}\n  - Attendees: %^{Attendees}, Suvrat
  - Date: %U\n  - Notes:\n    + %?\n  - Action items [/]\n    + [ ] "
         :prepend t)

        ("M" "Movie" entry (file suv-org-movies-file)
         "* TODO %^{Description}"
         :immediate-finish t)

        ("w" "Work task" entry (file suv-org-work-file)
         "* TODO %^{Type|TODO|DEP|BUG}-%^{Ticket number} - %^{Description}
  :PROPERTIES:
  :LINK:     https://helpshift.atlassian.net/browse/%\\1-%\\2
  :END:
  :LOGBOOK:\n  - Added - %U\n  :END:\n  ")

        ("r" "Reading list item" entry (file suv-org-reading-list-file)
         "* TODO %^{Description}\n  :LOGBOOK:\n - Added: %U\n  :END:
  %(current-kill 0)\n  %?")))

;; Show how you can know all of this from inside of Emacs. (`C-h v`)

;; Use `%c` instead of `%(current-kill 0)`

;;; org-capture-demo.el ends here
{% endhighlight %}

## Conclusion

Org capture is a great way of capturing information in a structured way with
minimal distraction from your workflow!
