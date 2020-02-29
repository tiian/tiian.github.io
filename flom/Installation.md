# Installation

Be sure you have GLIB2 installed on your system (Ubuntu verification command):

    dpkg --list | grep libglib2.0-dev

Download the tarball from https://sourceforge.net/projects/flom/files/ and expand it:

    tar xvzf flom-X.Y.Z.tar.gz

or pick-up flom source code (last commit) using git:

    git clone git://git.code.sf.net/p/flom/code flom

Configure & make:

    cd flom-X.Y.Z    
    ./configure
    make

Install:

    sudo make install

*flom* command is now installed in */usr/local/bin* directory:

    ls -la /usr/local/bin/flom
    -rwxr-xr-x 1 root root 161625 2013-12-20 22:41 /usr/local/bin/flom

add */usr/local/bin* directory to your search path (many times it's already OK), then you are ready to run *flom*:

    flom --version
    FLOM: Free LOck Manager
    Copyright (c) 2013-2020, Christian Ferrari; all rights reserved.
    License: GPL (GNU Public License) version 2
    Package name: flom; package version: 1.5.0
    Access https://github.com/tiian/flom for project community activities
    Documentation is available at http://www.tiian.org/flom/

# Tested platforms

**FLoM** is *free software* and it does **not** have a strict certification process, but I perform some tests before releasing a tarball.
Test log is available in file [TestLog](https://github.com/tiian/flom/blob/master/TestLog) inside root tarball directory.

To guarantee minimum software requirements, FLoM is primarily developed with Ubuntu 10.04, x86 (64 bit), virtual.

If you used a distribution not too far from the tested ones, you should succeed in *flom* installation.
If you found any issue, please send me a feedback using [Issues](https://github.com/tiian/flom/issues) or [Discussion](https://sourceforge.net/p/flom/discussion/)
