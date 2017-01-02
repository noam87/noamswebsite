+++
date = "2013-04-06T17:14:21-05:00"
title = "Using The Same bashrc/zshrc Across Computers"
draft = true
+++

Here is a simple method I use to share the same `.bashrc` / `.zshrc` /
`.bash_profile` on multiple computers, while still retaining unique
settings where I need them.

Suppose you want some special setting to apply only to your linux laptop, but
not to your mac laptop.

```
$ mkdir ~/configs
$ touch ~/.is_linux
```

Next:

```
if [ -f '.is_linux' ]; then
    echo "This message only shows on my Linux laptops!"
fi
```

Now with different config files you can configure different environments from
is single universal `*rc` file you keep in a
[dotfiles](https://github.com/noam87/DOTfiles) repo.

That's it. It's not fancy, but it works.
