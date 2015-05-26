===
git
===

Aliases::

    [alias]
        fp = format-patch --full-index
        lod = log --oneline --decorate
        lol = log --oneline --graph --decorate
        lou = log --oneline --graph --decorate --branches --not --remotes
        loggr = log --grep
        last = log --oneline --graph --decorate -1
        really-clean = clean -fdx
        st = status -sb
        ci = commit
        co = checkout
        ls-not-added = ls-files --others --exclude-standard
        ls-staged = diff --cached --name-status
        branches = branch -vv

Including local config overrides::

  [include]
        path = ~/.gitconfig-local

