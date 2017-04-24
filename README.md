apt-borrow
==========

Sometimes you have to install packages that you don't need permanently (e.g.
build dependencies). So you might want to not keep them around, eventually
forgetting why you installed them in the first place, while they waste disk
space and bandwith/time during updates, and add to the complexity of your setup.

The obvious solution is to just remove those packages afterwards (or to mark them
as automatically installed right away so that you don't forget about it).
However, if you don't carefully check if any of these packages were already (manually)
installed before, you likely end up uninstalling packages that are still needed.

`apt-borrow` to the rescue, a simple wrapper around `apt-get install` and
`apt-mark`, which installs the given packages and marks them as automatically
installed but only if they weren't already manually installed before. So next
time you run `apt[-get] autoremove` the new packages will be uninstalled (along
any other automatically installed package that isn't required anymore).

Installation
------------

```
add-apt-repository ppa:s.noack/ppa
apt-get update
apt-get install apt-borrow
```

Usage
-----

Lets say we need `git`, `cmake` and `gcc`. We neither know nor care whether any
of these are already installed. All we want is making sure that we have all of
them installed, and easily get rid again of any package which we didn't have before.

```
$ apt-borrow git cmake gcc
...
gcc is already the newest version (4:6.3.0-1).
gcc set to manually installed.
git is already the newest version (1:2.11.0-2).
...
The following NEW packages will be installed:
  cmake cmake-data libjsoncpp1
...
cmake set to automatically installed.
gcc set to automatically installed.
```

* `gcc` has been automatically installed before (as dependency of
  `build-essential`). If an update would be available it would have been
  installed now (but the package was already on the newest version).
  In additon, `apt[-get] install` (called by `apt-borrow`) marks any
  given package as manually installed. But `apt-borrow` marked it as
  automatically installed again.
* `git` has been installed before as well. But it was marked as manually
  installed, which `apt-borrow` didn't change.
* `cmake` hasn't been installed before. So it got installed now. But unlike
  `apt[-get] install` does, `apt-borrow` marked it as automatically installed.

When you run `apt[-get] autoremove`, it uninstalls any packages that are
marked as automatically installed and aren't required in order to satisfy
dependencies of manually installed packages, which also includes new packages
installed through `apt-borrow`. (Note that the `--purge` option below is
optional. It causes the packages to be completely removed, including their
configuration files and entry in `dpkg`.)

```
$ apt-get autoremove --purge
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  cmake* cmake-data* libjsoncpp1*
0 upgraded, 0 newly installed, 3 to remove and 6 not upgraded.
After this operation, 22.6 MB disk space will be freed.
Do you want to continue? [Y/n]
```
