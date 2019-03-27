FROM ubuntu:18.04

# Adapted from https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile

RUN apt-get update
RUN apt-get install -y mercurial libexpat1-dev git-core build-essential bison \
    flex libssl-dev libdb-dev libpcap-dev libc6-dbg libsysfs-dev gawk indent \
    pkg-config autoconf automake sudo ccache libsaxonb-java openjdk-8-jre-headless bc wget && rm -rf /var/lib/apt/lists/*

RUN useradd -ms /bin/bash ns3dce && adduser ns3dce sudo && echo -n 'ns3dce:ns3dce' | chpasswd

# Enable passwordless sudo for users under the "sudo" group
RUN sed -i.bkp -e \
      's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' \
      /etc/sudoers

USER ns3dce
WORKDIR /home/ns3dce/

ENV PATH /usr/lib/ccache:${PATH}

# Try original bake
RUN git clone https://gitlab.com/nsnam/bake.git && mkdir dce-linux-dev
WORKDIR dce-linux-dev

RUN ./../bake/bake.py configure -e dce-linux-dev && ./../bake/bake.py download -vvv 2>&1 | tee ../ubuntu18_dce-dev-download.txt && ./../bake/bake.py show 2>&1 | tee ../ubuntu18_dce-dev-show.txt
RUN ./../bake/bake.py build -vvv 2>&1 | tee ../ubuntu18_dce-dev-build.txt
RUN cd source/ns-3-dce && ./test.py -vvv 2>&1 | tee ../../ubuntu18_dce-dev-test.txt
# Only 3 out of 47 tests pass, further testing on local machine. Debugging glibc issues