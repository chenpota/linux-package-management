Show you how to package a .deb file in Linux step by step.

# Test environment
* Docker image: ubuntu:16.04

# Necessary packages
    root@9a7417b92dfe:/# apt-get install dh-make build-essential fakeroot devscripts

# Setting up shell environment variables
    root@9a7417b92dfe:/# DEBEMAIL="test@test.com"
    root@9a7417b92dfe:/# DEBFULLNAME="Tester"
    root@9a7417b92dfe:/# export DEBEMAIL DEBFULLNAME

# Sample
    dpkg-test/
    `-- hello-world
        |-- Makefile
        `-- main.c

```c
// main.c
#include <stdio.h>

int main()
{
    printf("hello world!!\n");
    return 0;
}
```

```makefile
# Makefile
DESTDIR=/
INSTALL_LOCATION=$(DESTDIR)/usr/
CFLAGS:=$(shell dpkg-buildflags --get CFLAGS)
LDFLAGS:=$(shell dpkg-buildflags --get LDFLAGS)

all: main.o
	cc $(CFLAGS) $(LDFLAGS) -o hello-world main.o

install:
	mkdir -p $(INSTALL_LOCATION)/bin
	cp hello-world $(INSTALL_LOCATION)/bin

clean:
	rm -f *.o hello-world
```
*DESTDIR* sees [[3]](http://www.gnu.org/prep/standards/html_node/DESTDIR.html#DESTDIR
)

# Prepare Debian packaging
    root@9a7417b92dfe:/dpkg-test/hello-world# dh_make -p hello-world_0.1 --createorig
    Type of package: (single, indep, library, python)
    [s/i/l/p]? s
    Email-Address       : test@test.com
    License             : blank
    Package Name        : hello-world
    Maintainer Name     : Tester
    Version             : 0.1
    Package Type        : single
    Date                : Fri, 30 Dec 2016 17:49:05 +0000
    Are the details correct? [Y/n/q] y
    Done. Please edit the files in the debian/ subdirectory now.
    root@9a7417b92dfe:/dpkg-test/hello-world# cd debian/
    root@9a7417b92dfe:/dpkg-test/hello-world/debian# rm -f *.ex *.EX README.*
    root@9a7417b92dfe:/dpkg-test/hello-world/debian# ls
    changelog  compat  control  copyright  hello-world-docs.docs  rules  source

## /dpkg-test/hello-world/debian/control

Provide the main meta data for the Debian package.

    Source: hello-world
    Priority: optional
    Maintainer: Tester <test@test.com>
    Build-Depends: debhelper (>=9)
    Standards-Version: 3.9.6
    
    Package: hello-world
    Architecture: any
    Depends: ${shlibs:Depends}, ${misc:Depends}
    Description: hello world
    
## /dpkg-test/hello-world/debian/changelog

Using "**dch -i**" in "**/dpkg-test/hello-world**" can edit "**/dpkg-test/hello-world/debian/changelog**".

    hello-world (0.1-1) unstable; urgency=medium
    
      * Initial release (Closes: #nnnn)  <nnnn is the bug number of your ITP>
    
     -- Tester <test@test.com>  Fri, 30 Dec 2016 17:49:05 +0000

## /dpkg-test/hello-world/debian/rules

Define how the Debian binary package is built.

# Generate a .deb file in /dpkg-test

    root@9a7417b92dfe:/dpkg-test/hello-world# debuild -us -uc
     dpkg-buildpackage -rfakeroot -D -us -uc
    dpkg-buildpackage: warning: using a gain-root-command while being root
    dpkg-buildpackage: source package hello-world
    dpkg-buildpackage: source version 0.1-1
    dpkg-buildpackage: source distribution unstable
    dpkg-buildpackage: source changed by Tester <test@test.com>
     dpkg-source --before-build hello-world
    dpkg-buildpackage: host architecture amd64
     fakeroot debian/rules clean
    dh clean 
       dh_testdir
       dh_auto_clean
    	make -j1 clean
    make[1]: Entering directory '/dpkg-test/hello-world'
    rm -f *.o hello-world
    make[1]: Leaving directory '/dpkg-test/hello-world'
       dh_clean
     dpkg-source -b hello-world
    dpkg-source: info: using source format '3.0 (quilt)'
    dpkg-source: info: building hello-world using existing ./hello-world_0.1.orig.tar.xz
    dpkg-source: info: building hello-world in hello-world_0.1-1.debian.tar.xz
    dpkg-source: info: building hello-world in hello-world_0.1-1.dsc
     debian/rules build
    dh build 
       dh_testdir
       dh_update_autotools_config
       dh_auto_configure
       dh_auto_build
    	make -j1
    make[1]: Entering directory '/dpkg-test/hello-world'
    cc -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2  -c -o main.o main.c
    cc -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wl,-Bsymbolic-functions -Wl,-z,relro -o hello-world main.o
    make[1]: Leaving directory '/dpkg-test/hello-world'
       dh_auto_test
     fakeroot debian/rules binary
    dh binary 
       dh_testroot
       dh_prep
       dh_auto_install
    	make -j1 install DESTDIR=/dpkg-test/hello-world/debian/hello-world AM_UPDATE_INFO_DIR=no
    make[1]: Entering directory '/dpkg-test/hello-world'
    mkdir -p /dpkg-test/hello-world/debian/hello-world/usr//bin
    cp hello-world /dpkg-test/hello-world/debian/hello-world/usr//bin
    make[1]: Leaving directory '/dpkg-test/hello-world'
       dh_installdocs
       dh_installchangelogs
       dh_perl
       dh_link
       dh_strip_nondeterminism
       dh_compress
       dh_fixperms
       dh_strip
       dh_makeshlibs
       dh_shlibdeps
       dh_installdeb
       dh_gencontrol
       dh_md5sums
       dh_builddeb
    dpkg-deb: building package 'hello-world' in '../hello-world_0.1-1_amd64.deb'.
     dpkg-genchanges  >../hello-world_0.1-1_amd64.changes
    dpkg-genchanges: warning: missing Section for source files
    dpkg-genchanges: warning: missing Section for binary package hello-world; using '-'
    dpkg-genchanges: including full source code in upload
     dpkg-source --after-build hello-world
    dpkg-buildpackage: full upload (original source is included)
    Now running lintian...
    warning: the authors of lintian do not recommend running it with root privileges!
    E: hello-world changes: bad-distribution-in-changes-file unstable
    W: hello-world source: no-section-field-for-source
    W: hello-world source: space-in-std-shortname-in-dep5-copyright <special license> (paragraph at line 5)
    W: hello-world source: out-of-date-standards-version 3.9.6 (current is 3.9.7)
    W: hello-world: wrong-bug-number-in-closes l3:#nnnn
    W: hello-world: new-package-should-close-itp-bug
    E: hello-world: changelog-is-dh_make-template
    E: hello-world: helper-templates-in-copyright
    W: hello-world: copyright-has-url-from-dh_make-boilerplate
    E: hello-world: copyright-contains-dh_make-todo-boilerplate
    E: hello-world: description-is-pkg-name hello world
    E: hello-world: extended-description-is-empty
    W: hello-world: no-section-field
    W: hello-world: binary-without-manpage usr/bin/hello-world
    Finished running lintian.
    root@9a7417b92dfe:/dpkg-test/hello-world# 

# Install .deb file to "/usr/bin" and execute hello-world

    root@9a7417b92dfe:/dpkg-test# dpkg -i hello-world_0.1-1_amd64.deb 
    Selecting previously unselected package hello-world.
    (Reading database ... 23521 files and directories currently installed.)
    Preparing to unpack hello-world_0.1-1_amd64.deb ...
    Unpacking hello-world (0.1-1) ...
    Setting up hello-world (0.1-1) ...
    root@9a7417b92dfe:/dpkg-test# hello-world
    hello world!!
    
# Reference
[^[1]] https://www.debian.org/doc/manuals/maint-guide/index.en.html

[^[2]] http://santi-bassett.blogspot.tw/2014/07/how-to-create-debian-package.html

[^[3]] http://www.gnu.org/prep/standards/html_node/DESTDIR.html#DESTDIR

[^[4]] https://wiki.debian.org/Hardening
