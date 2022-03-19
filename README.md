# Emacs Lisp Guide

### Table of Contents

- [Emacs Lisp Guide](#emacs-lisp-guide)
    - [Table of Contents](#table-of-contents)
  - [Audience](#audience)
  - [Programming in Emacs Lisp](#programming-in-emacs-lisp)
  - [After reading this guide](#after-reading-this-guide)
  - [Trivial basics](#trivial-basics)
  - [Evaluation](#evaluation)
  - [Discoverability](#discoverability)
    - [Finding functions of keybindings](#finding-functions-of-keybindings)
    - [Getting documentation](#getting-documentation)
    - [Find all bindings in the current buffer](#find-all-bindings-in-the-current-buffer)
    - [Searching for documentation topics](#searching-for-documentation-topics)
    - [Jumping to definition](#jumping-to-definition)
    - [Describe functions](#describe-functions)
  - [Basic concepts](#basic-concepts)
    - [Buffers](#buffers)
    - [Buffer-local variables](#buffer-local-variables)
    - [Project-wide buffer-local variables](#project-wide-buffer-local-variables)
    - [The point](#the-point)
    - [The region](#the-region)
    - [Text properties](#text-properties)
  - [Debugging](#debugging)
  - [Editing](#editing)
    - [Paredit](#paredit)
      - [Navigating](#navigating)
      - [Killing](#killing)
      - [Raising](#raising)
      - [Wrapping](#wrapping)
      - [Splitting](#splitting)
  - [Manipulating the buffer](#manipulating-the-buffer)
    - [Text properties](#text-properties-1)
  - [Navigating the buffer](#navigating-the-buffer)
    - [Save excursion](#save-excursion)
  - [Querying the buffer](#querying-the-buffer)
  - [Temporary buffers](#temporary-buffers)
  - [Defining interactive functions](#defining-interactive-functions)
  - [Defining your own major mode](#defining-your-own-major-mode)
  - [Defining a minor mode](#defining-a-minor-mode)
  - [Markers](#markers)
  - [Overlays](#overlays)
  - [Standard practices](#standard-practices)
    - [Namespacing](#namespacing)
  - [Alternative sources](#alternative-sources)

## Audience

Programmers who are too busy to read through long tutorials and
manuals, but who want to extend their editor. You don't need to learn
everything from the ground up, just enough knowledge to be
self-sufficient. You've been using Emacs for a while and now it's time
you started making some handy extensions for yourself.

There are a bunch of existing guides, but they don't strike the right
balance of useful and helpful. Some just list functions, others try to
explain Emacs Lisp from the ground up as a language. You don't need to
know everything right away. See the [Alternative sources](#alternative-sources)
section for a list of these.

## Programming in Emacs Lisp

I'm not going to explain the Emacs Lisp language itself in any
detail. Programming in
[Emacs Lisp](http://en.wikipedia.org/wiki/Emacs_Lisp) (look at the
Wikipedia page for the academic details) is similar to programming in
Python, Scheme, Common Lisp, JavaScript, Ruby, and languages like
that. Its syntax is funny but otherwise it's an imperative language
with similar data structures.

***One important difference*** compared to usual languages to be aware
of is that it has _dynamic scope_ by default. See
[Dynamic Binding](https://www.gnu.org/software/emacs/manual/html_node/elisp/Dynamic-Binding.html#Dynamic-Binding)
in the manual for the details. Almost all Emacs Lisp code you come
across today will be using
this. [Lexical Binding](https://www.gnu.org/software/emacs/manual/html_node/elisp/Lexical-Binding.html#Lexical-Binding)
has recently been added to Emacs, it will take a while for this to
permeate.

Like all Lisps, Emacs Lisp has macros which you can [read about in the
manual at your leisure.](https://www.gnu.org/software/emacs/manual/html_node/elisp/Macros.html#Macros)

## After reading this guide

The best, most comprehensive resource on Emacs Lisp is
[the manual](https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html). I
will reference this manual throughout this guide. I will not repeat
what's already there. You can reference this manually in a random
access fashion when you need to solve a problem.

I reference the manual throughout the guide by HTML link, but you can
read it inside your Emacs itself. Run: `C-h i m Elisp RET`

## Trivial basics

These are the basics to syntax that you can lookup in any guide or
just by looking at some Emacs Lisp code. I am assuming you're a
programmer who can pick things up like this just by looking at code. I
include these because I use them later:

``` lisp
(* 2 3)
(concat "a" "b")
(defun func (arg1 arg2)
  "Always document your functions."
   <function body>)
(defvar var-name <the value>
  "Always document your variables.")
(let ((x 1)
      (y 2))
  ...)
```

In Lisp the normal `LET` doesn't let you refer to previous variables,
so you need to use `LET*` for that. This is likely to trip people up,
so I include it here.

``` lisp
(let* ((x 1)
       (y x))
  ...)
```

To do many things at once in one expression, use `PROGN`:

``` lisp
(progn do-this
       do-that)
```

[See manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Local-Variables.html#Local-Variables)
for details.

The way to set variables is not obvious:

``` lisp
(setq var-name value)
```

Equality and comparison operators:

* `(eq major-mode 'a)`
* `(= 0 1)`
* `(> 0 1)`
* `(string= "a" "b")`
* `(string> "a" "b")`

Emacs Lisp has a bunch of equality operators. See
[the manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Equality-Predicates.html)
for gory details.

Data structures available: lists, vectors, rings, hashtables. Look them up in
[the manual](https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html).

## Evaluation

* Use `M-:` to evaluate any Emacs Lisp expression and print the
  result. I personally use this constantly.
* Use `C-x C-e` to evaluate the previous s-expression in the
  buffer. I personally never use this. See next binding.
* Use `C-M-x` to evaluate the current top-level s-expression. I use
  this to re-apply `defvar` and `defun` declarations.
* There is a REPL available by `M-x ielm`. I tend to use `M-:` rather
  than the REPL but you might like it.
* Use `M-x eval-buffer` to evaluate the whole buffer of Emacs Lisp
  code.

## Discoverability

A very important thing as an Emacs Lisp programmer is being able to
get the information you want in a few keystrokes. Here's a list of
ways to find what you need when you're writing Elisp code.

### Finding functions of keybindings

Find the function called by a keybinding: `C-h k`

This will show something like:

    C-p runs the command previous-line, which is an interactive compiled
    Lisp function in `simple.el'.

    It is bound to C-p.

    (previous-line &optional ARG TRY-VSCROLL)

You can click the link `simple.el` to go directly to the definition of
that function. Very handy indeed.

### Getting documentation

Functions and variables are distinguished in Emacs Lisp, so there are
two commands to do lookups:

* Run `C-h f` to show documentation for a function. This also works
  for macros.
* Run `C-h v` to show documentation for a variable.

You'll see something like:

    mapcar is a built-in function in `C source code'.

    (mapcar FUNCTION SEQUENCE)

    Apply FUNCTION to each element of SEQUENCE, and make a list of the results.
    The result is a list just as long as SEQUENCE.
    SEQUENCE may be a list, a vector, a bool-vector, or a string.

### Find all bindings in the current buffer

Run `C-h b` to show a massive list of keybindings and the command they
run. You'll see something like, e.g. in `markdown-mode`:

    C-c C-x d       markdown-move-down
    C-c C-x l       markdown-promote
    C-c C-x m       markdown-insert-list-item

### Searching for documentation topics

Use the commands called `apropos`.

* `M-x apropos`
* `M-x apropos-command`
* `M-x apropos-library`
* `M-x apropos-documentation`

### Jumping to definition

Install this package:
[elisp-slime-nav](https://github.com/purcell/elisp-slime-nav)

Now you can use `M-.` to jump to the identifer at point and `M-,` to
jump back.

### Describe functions

The range of `M-x describe-` functions are useful:

* `M-x describe-mode` (aka `C-h m`)
* `M-x describe-face`

Other ones have been mentioned above as keybindings.

## Basic concepts

### Buffers

All Emacs Lisp code when run has a current buffer. Operations that
claim to work on "the buffer" work on this current buffer. Some handy
functions, which you can run `C-h f` on to get more info:

* `(current-buffer)` - get the current buffer.
* `(with-current-buffer buffer-or-name ...)` - temporarily use the given buffer.
* `(set-buffer buffer-or-name)` - set the current buffer without
  switching to it.
* `(switch-to-buffer name)` - switch to the buffer visually.

See
[Buffers](https://www.gnu.org/software/emacs/manual/html_node/elisp/Buffers.html#Buffers)
in the manual for detailed info.

### Buffer-local variables

Buffers have local variables, for example:

* major-mode

You can use this variable to see what mode you're in, if you need it.

If you want to set your own buffer-local variable, use this:

``` lisp
(defvar your-variable-name nil "Your documentation here.")
```

Then later on in your code that will run in a given buffer, use:

``` lisp
(set (make-local-variable 'your-variable-name) <the-value>)
```

This is very handy in many scenarios when writing functionality. Note
that buffer local variables are reset when you revert the buffer or
change modes.

See
[manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Buffer_002dLocal-Variables.html#Buffer_002dLocal-Variables)
for details.

### Project-wide buffer-local variables

A handy way to set a buffer local variable for every file that's
within a directory structure is to use
[a `.dir-locals.el` file.](https://www.gnu.org/software/emacs/manual/html_node/emacs/Directory-Variables.html)

``` lisp
((nil . ((indent-tabs-mode . t)
         (fill-column . 80)))
 (c-mode . ((c-file-style . "BSD")
            (subdirs . nil)))
 ("src/imported"
  . ((nil . ((change-log-default-name
              . "ChangeLog.local"))))))
```

### The point

All Emacs Lisp code has a current point in the current buffer. It's a
number. It refers to where the cursor is. See
[the manual entry for point](http://www.gnu.org/software/emacs/manual/html_node/elisp/Point.html),
but here's the basics:

* `(point)` - current point
* `(point-max)` - maximum point of the buffer
* `(point-min)` - minimum point of the buffer (why is this not just `0`?
  Because of
  [narrowing](http://www.gnu.org/software/emacs/manual/html_node/elisp/Narrowing.html#Narrowing)).

### The region

Sometimes the region can be active, and you can use it in your Emacs
Lisp code to manipulate text specially. See
[the manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/The-Region.html#The-Region)
for details. Rundown:

* `(region-beginning)` - beginning of the region (a point)
* `(region-end)` - end of the region (a point)
* `(use-region-p)` - whether to try to use region-beginning/region-end
  for manipulation. Handy for use in commands.
* `(region-active-p)` - also handy to know whether the region is
  active.

Here's an command that uses some region functions:

``` lisp
(defun print-upper-region ()
  "Demo to print the uppercased version of the active region."
  (interactive)
  (when (region-active-p)
    (message "%S" (let ((string (buffer-substring (region-beginning)
                                                  (region-end))))
                    (with-temp-buffer
                      (insert string)
                      (upcase-region (point-min)
                                     (point-max))
                      (buffer-substring-no-properties (point-min)
                                                      (point-max)))))))
```

To run it, `C-M-x` it, select some text and run `M-x print-upper-region`.

### Text properties

When you manipulate text in Elisp, it can have properties applied to
it, and those properties can be queried. Full details are
[here](http://www.gnu.org/software/emacs/manual/html_node/elisp/Text-Properties.html#Text-Properties)
but see the "Manipulating the buffer" section in this guide for examples.

## Debugging

Run `M-: (setq debug-on-error t) RET` and any errors will open up the
debugger.

I'll write more about using the debugger stepper and breakpoints later.

## Editing

### Paredit

Install and enable
[paredit](http://www.emacswiki.org/emacs/ParEdit). Nobody sane writes
Lisp without paredit (or its shiny cousin,
[smartparens](https://github.com/Fuco1/smartparens);
or its evil twin,
[lispy](https://github.com/abo-abo/lispy)). You will never
have unbalanced parentheses, brackets, braces, or strings. Learn to
accept this and you will enjoy this mode.

As discussed in the discoverability section, use `C-h f paredit-mode
RET` to see the documentation for this mode.

Learn the following helpful keybindings:

#### Navigating

* `C-M-u` - Go up a node.
* `)` - Go to the end of the node or the end of the parent node when repeated.
* `C-M-f` - Go to the end of the node.
* `C-M-b` - Go to the start of the node.

#### Killing

`C-k` - Kill everything from here to the end of the line, including
  any following lines that are included in the scope of the nodes
  being killed. It will also kill inside strings but stop at the end
  of the string.

#### Raising

`M-r` - Replace the parent node by the current node.

    (|foo) -> foo
    (foo |bar mu) -> bar
    (foo (bar |mu zot) bob) -> (foo mu bob)

#### Wrapping

* `C-M-(` to wrap the following node in parens.
* Alternatively, `C-M-SPC` to select the whole node, or just use your
  normal region selection and run `(` or
  `[` or `{` to wrap that selection.

#### Splitting

* `M-s` to split the current node. This works on parenthesized expressions or strings.
* `M-J` to join two nodes. Works same as above in reverse.

## Manipulating the buffer

These are the most common:

* `(insert "foo" "bar")` - to insert text at point.
* `(delete-region start end)` - to delete the region of text.
* `(insert-buffer-substring-no-properties buffer start end)` - insert text from another buffer.
* `(insert-file-contents <filename>)` - insert from a file.

Any other command that inserts things can be called from Emacs Lisp, too.

### Text properties

To add properties to text in the buffer, use:

``` lisp
(put-text-property start end 'my-property-name <value>)
```

To completely reset the properties of text to just this, use:

``` lisp
(set-text-properties start end 'my-property-name <value>)
```

To retrieve properties back from the text, use:

``` lisp
(get-text-property <point> 'my-property-name)
```

To propertize a string before it's inserted into a buffer, use:

``` lisp
(propertize "hello" 'my-property-name <value> 'another-prop <value2>)
```

## Navigating the buffer

Here are the common ones:

* `(goto-char <point>)` - go to the point.
* `(forward-char n)` - go forward n chars. Accepts a negative argument.
* `(end-of-line)` - self-explanatory.
* `(beginning-of-line)` - self-explanatory.
* `(skip-chars-forward "chars")` - skip given chars.
* `(skip-chars-backward "chars")` - skip given chars back.
* `(search-forward "foo")` - search for foo, move cursor there.
* `(search-backward "foo")` - search backward.
* `(search-forward-regexp "blah")` - same, but with regexes.
* `(search-backward-regexp "blah")` - same, but with regexes.

If there's a kind of navigation you want to do that you don't know the function name for, think of how you would do it with your keyboard and then use `C-h k` on the commands to find out the functions being run.

### Save excursion

Often you want to jump around the buffer to either query or manipulate something, and then go back to where you were originally. To do this, use:

``` lisp
(save-excursion ...)
```

For example:

``` lisp
(save-excursion (beginning-of-line) (looking-at "X"))
```

Will return whether the current line starts with `X`.

Similarly there is `save-window-excursion`.

## Querying the buffer

* `(buffer-substring start end)` - get the string at point, including text properties.
* `(buffer-substring-no-properties start end)` - get the string at point, excluding text properties.
* `(buffer-string)` - return the string of the whole buffer.
* `(looking-at "[a-zA-Z]+")` - does text following point match the regex?
* `(looking-back "[a-zA-Z]+")` - does text preceding point match the regex?

## Temporary buffers

It's often useful to do some work in a temporary buffer so that you
can use your normal Elisp code to generate a string and some
properties, for example:

``` lisp
(with-temp-buffer
  (insert "Hello!"))
```

## Defining interactive functions

To be able to run a function of your own from a keybinding, it needs
to be interactive. You need to add `(interactive)` to your `defun`:

``` lisp
(defun foo ()
  "Some function."
  (interactive)
  (do-some-stuff))
```

There's a bunch of variations for `INTERACTIVE`,
[see the manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Using-Interactive.html).

Now your function `foo` is interactive, you can use it in a
keybinding:

``` lisp
(define-key emacs-lisp-mode-mode (kbd "C-c C-f") 'foo)
```

## Defining your own major mode

You can generally use `define-derived-mode`. See
[the manual on this.](http://www.gnu.org/software/emacs/manual/html_node/elisp/Derived-Modes.html)

Example:

``` lisp
(define-derived-mode hypertext-mode
   text-mode "Hypertext"
   "Major mode for hypertext.
 \\{hypertext-mode-map}"
   (setq case-fold-search nil))

(define-key hypertext-mode-map
   [down-mouse-3] 'do-hyper-link)
```

## Defining a minor mode

Minor modes act as enhancements to existing modes. See
[the manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Defining-Minor-Modes.html)
about `define-minor-mode`.

A dummy example:

``` lisp
(defvar elisp-guide-mode-map (make-sparse-keymap))
(define-minor-mode elisp-guide-mode "A simple minor mode example."
  :lighter " ELGuide"
  :keymap elisp-guide-mode-map
  (if (bound-and-true-p elisp-guide-mode)
      (message "Elisp guide activated!")
    (message "Bye!")))
(define-key elisp-guide-mode-map (kbd "C-c C-a") 'elisp-guide-go)
(defun elisp-guide-go ()
  (interactive)
  (message "Go!"))
```

Run `M-x elisp-guide-mode` to activate it and run it again to disable it.

Real examples of minor modes:

* [structured-haskell-mode](https://github.com/chrisdone/structured-haskell-mode/blob/master/elisp/shm.el#L110)
* [paredit-mode](https://github.com/emacsmirror/paredit/blob/master/paredit.el#L203)
* [god-mode](https://github.com/chrisdone/god-mode/blob/master/god-mode.el#L80..L86)

## Markers

Markers are handy objects that store a point, and changes to the
buffer make the marker position move along. See
[the manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/Markers.html),
which has a good section explaining it. Their use-case is probably
more intermediate than for a tutorial like this, so I include them
only so that you're aware of them.

Here's an example:

``` lisp
(defun my-indent-region (beg end)
  (interactive "r")
  (let ((marker (make-marker)))
    (set-marker marker (region-end))
    (goto-char (region-beginning))
    (while (< (point) marker)
      (funcall indent-line-function)
      (forward-line 1))))
```

You need to store the end of the region before you start changing the
buffer, because the integer position will increase as you start
indenting lines. So you store it in a marker and that marker's value
updates as the buffer's contents changes.

## Overlays

See
[the manual on overlays](http://www.gnu.org/software/emacs/manual/html_node/elisp/Overlays.html),
these are a handy tool for a special kind of text that behaves as if
separate and above the buffer. This is more advanced, by the time you
want to use overlays you'll be happy reading the manual entry about it.

## Standard practices

### Namespacing

Emacs Lisp doesn't support modules. We go by convention. If your
module name is `foo`, then name all your top-level bindings by
prefixing it with `foo-`. Example:

    (defun foo-go ()
      "Go!"
       ...)

    (provide 'foo)

To make this easier on your fingers, you can use something like:

``` lisp
(defun emacs-lisp-expand-clever ()
  "Cleverly expand symbols with normal dabbrev-expand, but also
if the symbol is -foo, then expand to module-name-foo."
  (interactive)
  (if (save-excursion
        (backward-sexp)
        (when (looking-at "#?'") (search-forward "'"))
        (looking-at "-"))
      (if (eq last-command this-command)
          (call-interactively 'dabbrev-expand)
        (let ((module-name (emacs-lisp-module-name)))
          (progn
            (save-excursion
              (backward-sexp)
              (when (looking-at "#?'") (search-forward "'"))
              (unless (string= (buffer-substring-no-properties
                                (point)
                                (min (point-max) (+ (point) (length module-name))))
                               module-name)
                (insert module-name)))
            (call-interactively 'dabbrev-expand))))
    (call-interactively 'dabbrev-expand)))

(defun emacs-lisp-module-name ()
  "Search the buffer for `provide' declaration."
  (save-excursion
    (goto-char (point-min))
    (when (search-forward-regexp "^(provide '" nil t 1)
      (symbol-name (symbol-at-point)))))
```

And then:

``` lisp
(define-key emacs-lisp-mode-map (kbd "M-/") 'emacs-lisp-expand-clever)
```

Now you can write `(defun -blah M-/` and get `(defun foo-blah`. You
need a `(provide 'foo)` line at the bottom of your file for this to work.

## Alternative sources

* http://steve-yegge.blogspot.it/2008/01/emergency-elisp.html
* http://lispp.wordpress.com/2009/11/25/emacs-lisp-cheatsheet/
* http://stackoverflow.com/questions/5238245/elisp-programming-whats-the-best-setup
* http://nic.ferrier.me.uk/blog/2012_07/tips-and-tricks-for-emacslisp
* https://www.gnu.org/software/emacs/manual/html_node/eintr/index.html
* http://www.emacswiki.org/emacs/EmacsLispIntro
* http://www.emacswiki.org/emacs/LearnEmacsLisp
* http://bzg.fr/learn-emacs-lisp-in-15-minutes.html
* http://www.delorie.com/gnu/docs/emacs-lisp-intro/emacs-lisp-intro_toc.html
* http://cjohansen.no/an-introduction-to-elisp
* http://emacswiki.org/emacs/ElispCookbook
* [Emacs cheah sheet](https://web.archive.org/web/20150525182115/http://wikemacs.org/wiki/Emacs_Lisp_Cheat_Sheet)
