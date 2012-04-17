# Commit messages

The first line of a commit should be 50 characters or less.  This is a soft limit, so that git's command line output is more readable.  Use this line as a clear summary of the change.  Try using imperative language in this line.  A good commit message is "Add reactive variable for CPU clock speed".  Worse examples include "cpu clock speed" or "Added Meteor.CPU_speed so that we can avoid a setTimeout in the main example."

Subsequent lines (if any) can be fairly verbose and detailed.  Use your judgement.

# Whitespace

* 2 space indents (setq js-indent-level 2)
* spaces not literal tabs (setq-default indent-tabs-mode nil)
* no trailing whitespace (setq-default show-trailing-whitespace t)

Emacs users should check out [js2-mode](https://github.com/mooz/js2-mode) for a nice way to avoid silly javascript errors, and help enforce standards.
