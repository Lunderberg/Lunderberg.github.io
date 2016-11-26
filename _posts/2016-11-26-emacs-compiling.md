---
layout: post
title: Compiling from emacs
tags: [c++, make, emacs]
---

Emacs can launch subprocesses, which can be used to compile code immediately.
This is built in with the `M-x compile` command.
It asks for the command to run, then runs it,
  saving the output in the `*compile*` buffer.
This is remarkably convenient,
  removing the need to switch between terminal and editor.

Furthermore, emacs has the ability to parse error messages.
With the ``C-x ` `` shortcut (`next-error`),
  emacs will jump to the location of the next error.
This opens files if needed, and jumps directly to the line that caused the error.
Repeating the command will step through all of the errors that came up.

I want to improve on this a bit.
Specifying the command to be run is very flexible,
  but 95% of the time, I just want to run `make`,
  running in some parent directory of the current file.
I want to make this common case be more convenient.

First up, we need to find the makefile.
With elisp, I make a more general function.
Given a regular expression and an initial directory,
  search upwards through the directory structure until you find a file that matches.

```lisp
(defun find-file-in-hierarchy (current-dir pattern)
  (let ((filelist (directory-files current-dir 'absolute pattern))
        (parent (parent-directory (expand-file-name current-dir))))
    (if (> (length filelist) 0)
        (car filelist)
      (if parent
        (find-file-in-hierarchy parent pattern)))))
```

With this, I can jump straight to the makefile.
I use GNU Make, so the makefile may be called
  `GNUmakefile`, `makefile`, or `Makefile`.
The regular expression `"^\\(GNUm\\|M\\|m\\)akefile$"`
  will match any of these.
Now, putting this all together into a function to run make.

```lisp
(defun find-makefile-compile ()
  (interactive)
  (let ((file (find-file-in-hierarchy
                 (file-name-directory buffer-file-name)
                 "^\\(GNUm\\|M\\|m\\)akefile$")))
    (if file
      (let ((directory (file-name-directory file)))
          (compile (format "cd \"%s\" && make" directory))))))
```

This can now be bound to a shortcut.
As an added bonus, we can make a prefix argument (`C-u`),
  which will run `make clean` instead.

```lisp
(defun find-makefile-compile (clean)
  (interactive "p")
  (let ((file (find-file-in-hierarchy
                 (file-name-directory buffer-file-name)
                 "^\\(GNUm\\|M\\|m\\)akefile$")))
    (if file
      (let ((directory (file-name-directory file)))
          (compile (format "cd \"%s\" && make %s"
                    directory (if (> clean 1) "clean" "")))))))
```