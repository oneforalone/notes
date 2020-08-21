# Creating a Debian Package

## Rebuilding a Package from its Sources

### Getting the sources and make the changes

~~~sh
$ apt source samba
$ cd samba-4.9.5-dfsg
$ dch --local falcot
~~~

Here, `dch` command is included in `devscripts` package.

Changes should update in `debian/rules` and `debian/control`:

* `debian/rule`: drives the steps in the package build process. In the simplest
case, the lines concerning the initial configuration (./configure ...) or the
actual build ($(MAKE) ... or make ...) are easy to spot. If these commands are
not explicitly called, the are probably to a side effect of another explict
command, in which case please refer to their documentation to learn more about
how to change the default behavior. With packages using `dh`, you might need to
add an override for the `dh_auto_configure` or `dh_auto_build` commands (see
their respective manual pages for expanations on how to archive this).

* `debian/control`: contains a description of the generated packages. In
particular, this file contains *Build-Depends* lines controlling the list of
dependencies that must be fulfilled at package build time. These often refer
to versions of packages contained in the distribution the source package comes
from, but which may not be available in the distribution used for the rebuild.
There is no automated way to determine if a dependency is real or only specified
to guarantee that the build should only be attempted with the latest version of
a library -- this is the only available way to force an _autobuilder_ to use a
given package version during build, which is why Debian maintainers frequently
use strictly versioned build-dependencies.

Reading document files called INSTALL will help you figure out the appropriate
dependencies.

### Staring the Rebuild

~~~sh
$ dpkg-buildpackage -us -uc
[...]
~~~

Before run this build command, you should install all develop version packages
of dependencies.


## Building your First Package

1. creating the package directory contianed the target source package file.
2. enter the directory and run `dh_make`:

~~~sh
$ mkdir hello-1.0
$ cd hello-1.0
$ dh_make --native

Type of package: (single, indep, library, python)
[s/i/l/d]? i

[...]
~~~

Define your **EMAIL** and **DEBFULLNAME** avoiding type them multiple times at
your `~/.bashrc`:

~~~shell
export EMAIL="mail-address@example.com"
export DEBFULLNAME="First-Name Last-Name"
~~~

All you need is modifing `control`, `rule` and `changelog` file under `debian`
directory. As said above, `control` contains the dependencies, and `rule` is
a alternative of Makefile, which controls the building steps. More details,
please refer to: [Required files under the debian directory](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html)
for English version, [debian 目录详情](https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html) for Chinese.
