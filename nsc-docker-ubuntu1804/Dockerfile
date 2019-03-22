FROM ubuntu:18.04

# Adapted from https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile

RUN apt-get update
RUN export DEBIAN_FRONTEND=noninteractive && apt-get install -y gcc g++ python python-dev mercurial python-setuptools git autoconf cvs bzr unrar gdb valgrind uncrustify doxygen graphviz imagemagick python-sphinx dia gsl-bin libgslcblas0 libgsl-dev flex bison libfl-dev tcpdump libxml2 libxml2-dev cmake libc6-dev libc6-dev-i386 libclang-dev llvm-dev automake vtun lxc libexpat1-dev git-core build-essential libssl-dev libdb-dev libpcap-dev libc6-dbg libsysfs-dev gawk indent pkg-config sudo ccache libsaxonb-java openjdk-8-jre-headless bc wget gcc-5 g++-5 gcc-5-multilib g++-5-multilib libgcc1-dbg libstdc++6-5-dbg && rm -rf /var/lib/apt/lists/*

RUN rm /usr/bin/gcc && rm /usr/bin/g++ && ln -s /usr/bin/gcc-5 /usr/bin/gcc && ln -s /usr/bin/g++-5 /usr/bin/g++

RUN useradd -ms /bin/bash ns3nsc && adduser ns3nsc sudo && echo -n 'ns3nsc:ns3nsc' | chpasswd

# Enable passwordless sudo for users under the "sudo" group
RUN sed -i.bkp -e \
      's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' \
      /etc/sudoers

USER ns3nsc
WORKDIR /home/ns3nsc/

ENV PATH /usr/lib/ccache:${PATH}

WORKDIR /home/ns3nsc/

# Building with scons does not work
# RUN hg clone https://secure.wand.net.nz/mercurial/nsc nsc-dev && cd nsc-dev
# RUN python2 scons.py 2>&1 | tee ubuntu18_nsc-scons.txt

# Building NSC with bake
RUN git clone https://gitlab.com/nsnam/bake.git && cd bake
RUN ./bake/bake.py configure -e ns-allinone-3.29 -e nsc-0.5.3
RUN ./bake/bake.py download -vvv 2>&1 | tee ubuntu18_nsc-download.txt && ./bake/bake.py show 2>&1 | tee ubuntu18_nsc-show.txt
RUN ./bake/bake.py build -vvv 2>&1 | tee ubuntu18_nsc-build.txt
