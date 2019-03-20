# NS3 DCE & NSC experiments
Trying to build DCE and/or NSC on different Linux distributions and versions. Mostly Ubuntu 16.04 and 18.04 for now.

## ns-3 + DCE
DCE is the "successor" of NSC, which is not supported anymore.
The DCE project page on nsnam, which is often linked to by Github projects and other documentation, yields a 404 www.nsnam.org/overview/projects/direct-code-execution/ (2019-03-12). 

Documentation for DCE:

https://www.nsnam.org/docs/dce/manual/html/getting-started.html

https://ns-3-dce.readthedocs.io/en/latest/getting-started.html


 * describes basic mode (DCE without Linux kernel) with bake
 * advanced mode (DCE with linux kernel) with waf
	-> build manually and then download net-next-sim (which includes DCE)
	-> probably https://github.com/direct-code-execution/net-next-sim (where sim-ns3-dev branch is no longer updated)
		-> they refer to https://github.com/libos-nuse/net-next-nuse (which seems to fail build)
		-> recommended build steps fail (https://github.com/libos-nuse/net-next-nuse/blob/nuse/Documentation/virtual/libos-howto.txt)

## Building DCE with bake
"Supported" Linux releases
    Ubuntu 10.04 64bit
    Ubuntu 12.04 32bit/64bit
    Ubuntu 12.10 64bit
    Ubuntu 13.04 64bit
    Ubuntu 13.10 64bit (new)
    Fedora 18 32bit
    CentOS 6.2 64bit
Apparently also possible with Ubuntu 16.04
### Packages
#### Needed for ns-3 
sudo apt install gcc g++ python python-dev mercurial python-setuptools git qt5-default gir1.2-goocanvas-2.0 python-gi python-gi-cairo python-pygraphviz python3-gi python3-gi-cairo python3-pygraphviz gir1.2-gtk-3.0 ipython ipython3 openmpi-bin openmpi-common openmpi-doc libopenmpi-dev autoconf cvs bzr unrar gdb valgrind uncrustify doxygen graphviz imagemagick texlive texlive-extra-utils texlive-latex-extra texlive-font-utils texlive-lang-portuguese dvipng python-sphinx dia gsl-bin libgslcblas0 libgslcblas0:i386 flex bison libfl-dev tcpdump sqlite sqlite3 libsqlite3-dev libxml2 libxml2-dev cmake libc6-dev libc6-dev-i386 libclang-dev llvm-dev automake libgtk2.0-0 libgtk2.0-dev
#### Additionally for DCE
(taken from https://www.nsnam.org/wiki/GSoC2018_DCE_Upgrade#Install_Direct_Code_Execution)
sudo apt install cmake cvs git bzr unrar p7zip-full autoconf build-essential bison flex g++ libc6 libc6-amd64 libdb-dev libexpat1-dev libpcap-dev libssl-dev python-dev python-pygraphviz python-setuptools qt4-dev-tools

#### Missing but optional according to bake.py download:
cxxfilt # pip install cxxfilt
pygoocanvas # not available for Ubuntu bionic beaver, drawing stuff
libc-debug # should be installed? (libclang-dev libc6-dbg)

Still missing python-pygoocanvas and libc-debug

### Build steps for DCE with Linux kernel (Mostly from https://ns-3-dce.readthedocs.io/en/latest/getting-started.html)



## ns-3 + NSC

TODO: add old documentation