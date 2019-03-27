FROM ubuntu:16.04

# Adapted from https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile

RUN apt-get update
RUN apt-get install -y gcc g++ python python-dev mercurial python-setuptools git autoconf cvs bzr unrar gdb valgrind uncrustify doxygen graphviz imagemagick python-sphinx dia gsl-bin libgsl2 libgsl-dev flex bison libfl-dev tcpdump libxml2 libxml2-dev cmake libc6-dev libc6-dev-i386 libclang-dev llvm-dev automake vtun lxc libexpat1-dev git-core build-essential libssl-dev libdb-dev libpcap-dev libc6-dbg libsysfs-dev gawk indent pkg-config sudo ccache libsaxonb-java openjdk-8-jre-headless bc wget && rm -rf /var/lib/apt/lists/*

RUN useradd -ms /bin/bash ns3nsc && adduser ns3nsc sudo && echo -n 'ns3nsc:ns3nsc' | chpasswd

# Enable passwordless sudo for users under the "sudo" group
RUN sed -i.bkp -e \
      's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' \
      /etc/sudoers

USER ns3nsc
WORKDIR /home/ns3nsc/

ENV PATH /usr/lib/ccache:${PATH}

# Building with scons does not work
# RUN hg clone https://secure.wand.net.nz/mercurial/nsc nsc-dev && cd nsc-dev
# RUN python2 scons.py 2>&1 | tee ubuntu16_nsc-scons.txt

# Building NSC with bake
RUN git clone https://gitlab.com/nsnam/bake.git && mkdir ns3-nsc
WORKDIR ns3-nsc

RUN ./../bake/bake.py configure -e nsc-dev && ./../bake/bake.py download -vvv 2>&1 | tee ubuntu16_nsc-download.txt \
	&& ./../bake/bake.py show 2>&1 | tee ubuntu16_nsc-show.txt
RUN ./../bake/bake.py build -vvv 2>&1 | tee ubuntu16_nsc-build.txt

# Download nstor and test NSC functionality
RUN cd .. && git clone https://github.com/crud3/nstor.git && cd nstor && git checkout bump-ns-3.27
WORKDIR /home/ns3nsc/nstor

# Because of https://github.com/crud3/nstor/issues/1, delete 01-single-star-web and just run tor-example
RUN rm -rf scratch/01-single-star-web
RUN ./waf clean ; CXXFLAGS="-Wall" ./waf configure --build-profile=debug --enable-examples --enable-tests --with-nsc=/home/ns3nsc/ns3-nsc/build/ && CXXFLAGS="-Wall" ./waf

# time $command does not work in Dockerfile, but works when in a shell in the container
# to execute manually: time ./waf --run "tor-example --use-nsc=true" | tee tor-example-nsc.txt
RUN ./waf --run "tor-example --use-nsc=true" | tee tor-example-nsc.txt

