---
title: "Spacemacs"
date: 2020-02-03T22:39:38+08:00
author: "Fralonra"
cover: ""
tags: ["spacemacs"]
keywords: ["spacemacs"]
description: "Some notes on Spacemacs"
showFullContent: false
---

# Spacemacs

### Installation

https://github.com/syl20bnr/spacemacs#install

```
git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
```

### Mirror

For example: https://mirror.tuna.tsinghua.edu.cn/help/elpa/

```lisp
;; .spacemacs dotspacemacs/user-init()
(setq configuration-layer--elpa-archives
    '(("melpa-cn" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/melpa/")
      ("org-cn"   . "http://mirrors.tuna.tsinghua.edu.cn/elpa/org/")
      ("gnu-cn"   . "http://mirrors.tuna.tsinghua.edu.cn/elpa/gnu/")))
```

### Layers

```lisp
dotspacemacs-configuration-layers
   '(
      html
      javascript
      markdown
      go
      helm
      auto-completion
      emacs-lisp
      (shell :variables
             shell-default-height 30
             shell-default-position 'bottom)
      syntax-checking
   )
```

##### syntax-checking

http://www.flycheck.org/en/latest/user/installation.html

- `SPC e v` flycheck-verify-setup
- `SPC u C-c ! x` enable checker

##### javascript

- Turn off js2-mode errors & warnings

```lisp
;; .spacemacs dotspacemacs/user-config
(setq js2-mode-show-parse-errors nil)
(setq js2-mode-show-strict-warnings nil)
```

### Keybindings

- neotree
`SPC f t`
- shell
`SPC '` shell layer
`:shell`
- insert new line
`SPC i j`
`] SPC` below
`[ SPC` above
