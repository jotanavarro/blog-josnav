---
title: "Emacs and MacOS"
date: 2021-01-01T19:52:09+01:00
tags: ["emacs", "gnu", "macos"]
---


I recently migrated from GNU/Linux to MacOS.  My favorite tool to edit text and code is Emacs, in particular I use the [Spacemacs](https://www.spacemacs.org) distribution and the evil mode.  There are several reasons why I enjoy using this tool, but I will not delve in them (for now).

But in MacOS, the first time we boot Emacs, we will be greeted by an error message claiming that `ls does not support --dired; see "dired-use-ls-dired" for more details.`.  In Emacs we can see the documentation of a variable with `C-h v`, so if we type the variable name there and we can read:

```
dired-use-ls-dired is a variable defined in ‘dired.el’.
Its value is nil
Original value was ‘unspecified’

  You can customize this variable.

Documentation:
Non-nil means Dired should pass the "--dired" option to "ls".
If nil, don’t pass "--dired" to "ls".
The special value of ‘unspecified’ means to check whether "ls"
supports the "--dired" option, and save the result in this
variable.  This is performed the first time ‘dired-insert-directory’
is invoked.

Note that if you set this option to nil, either through choice or
because your "ls" program does not support "--dired", Dired
will fail to parse some "unusual" file names, e.g. those with leading
spaces.  You might want to install ls from GNU Coreutils, which does
support this option.  Alternatively, you might want to use Emacs’s
own emulation of "ls", by using:
  (setq ls-lisp-use-insert-directory-program nil)
  (require 'ls-lisp)
This is used by default on MS Windows, which does not have an "ls" program.
Note that ‘ls-lisp’ does not support as many options as GNU ls, though.
For more details, see Info node ‘(emacs)ls in Lisp’.
```

Which tells us that by default Emacs expects `ls` to support `--dired`.  And sadly in MacOS:

```
jose@almanzora ~ % ls --dired
ls: illegal option -- -
usage: ls [-@ABCFGHLOPRSTUWabcdefghiklmnopqrstuwx1%] [file ...]
```

So according to the documentation above, we can do "like on MS Windows" and use `ls-lisp` emulation, or use the GNU `coreutils`.

#### Using _ls-lisp_ emulation

You can tell Emacs to behave like it does in _Windows_ by appending this to your configuration file:

```lisp
(setq ls-lisp-use-insert-directory-program nil)
(require 'ls-lisp)
```

Personally, I would *not* recommend this and I would go with the next solution, unless we do not want to use the `coreutils` for some reason.

#### Using GNU _coreutils_

Setting `dired-use-ls-dired` to anything but `nil` will make Emacs pass `--dired` to `ls`.

You can install the GNU implementation of the `coreutils` from homebrew like this:

```
brew install coreutils
```

This is quite neat, as it will install all the coreutils from GNU and add them to your `PATH` prefixed by the letter `g`.  For example the _GNU ls_ will be `gls` and the _MacOS ls_ will just be `ls`.  It is easy to verify with a simple `which`

```
jose@almanzora ~ % which ls
/bin/ls
jose@almanzora ~ % which gls
/usr/local/bin/gls
```

Now I must introduce the variable `insert-directory-program` which tells Emacs which program to use for listing the directories.  This can be found digging through the documentation (Again, `C-h v`) of `dired.el`.

So now we know how to tell Emacs to pass `--dired` to `ls` and how to tell it to use any arbitrary program for this, so we must append this to our configuration file `dotspacemacs`:

```lisp
(setq dired-use-ls-dired t
      insert-directory-program "/usr/local/bin/gls")
```

If we use the same configuration on different system families, we can encapsulate the previous block with a `when` that only applies those variables for _MacOS_ like this:

```lisp
(when (string= system-type "darwin")
  (setq dired-use-ls-dired t
        insert-directory-program "/usr/local/bin/gls"))
```

And with this, we will not break our configuration for `GNU/Linux` systems!
