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

RUN mkdir -p /home/ns3dce/dce-linux-dev
WORKDIR /home/ns3dce/dce-linux-dev
# Try original bake
RUN hg clone http://code.nsnam.org/bake

RUN ./bake/bake.py configure -e dce-linux-dev
# -e dce-umip-dev
RUN ./bake/bake.py download -vvv 2>&1 | tee ubuntu18_dce-dev-download.txt && ./bake/bake.py show 2>&1 | tee ubuntu18_dce-dev-show.txt
RUN ./bake/bake.py build -vvv 2>&1 | tee ubuntu18_dce-dev-build.txt
RUN cd source/ns-3-dce && ./test.py -vvv 2>&1 | tee ubuntu18_dce-dev-test.txt
