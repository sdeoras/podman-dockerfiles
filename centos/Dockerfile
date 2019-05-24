FROM centos

RUN yum install -y \
  atomic-registries \
  btrfs-progs-devel \
  containernetworking-cni \
  device-mapper-devel \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  go \
  golang-github-cpuguy83-go-md2man \
  gpgme-devel \
  iptables \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  make \
  ostree-devel \
  pkgconfig \
  runc \
  containers-common \
  which \
  wget

# install golang
RUN mkdir -p /Downloads
WORKDIR /Downloads
RUN wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
RUN tar -zxvf go1.12.5.linux-amd64.tar.gz
RUN mv go /usr/local
RUN ln -s /usr/local/go/bin/* /usr/local/bin
RUN mkdir /go
RUN rm -rf /Downloads
ENV GOPATH=/go
RUN go get github.com/cpuguy83/go-md2man

# build ostree
RUN git clone https://github.com/ostreedev/ostree /ostree
WORKDIR /ostree
RUN git submodule update --init
RUN yum install -y automake bison e2fsprogs-devel fuse-devel libtool xz-devel zlib-devel
RUN ./autogen.sh --prefix=/usr --libdir=/usr/lib64 --sysconfdir=/etc
# remove --nonet option due to https:/github.com/ostreedev/ostree/issues/1374
RUN sed -i '/.*--nonet.*/d' ./Makefile-man.am
RUN make
RUN make install
WORKDIR /

# build conmon
RUN git clone https://github.com/containers/conmon
WORKDIR /conmon
RUN make
RUN install -D -m 755 bin/conmon /usr/libexec/podman/conmon
WORKDIR /

# build runc
RUN git clone https://github.com/opencontainers/runc.git $GOPATH/src/github.com/opencontainers/runc
WORKDIR $GOPATH/src/github.com/opencontainers/runc
RUN make BUILDTAGS="selinux seccomp"
RUN cp runc /usr/bin/runc
WORKDIR /

# build network plugins
RUN git clone https://github.com/containernetworking/plugins.git $GOPATH/src/github.com/containernetworking/plugins
WORKDIR $GOPATH/src/github.com/containernetworking/plugins
RUN ./build_linux.sh
RUN mkdir -p /usr/libexec/cni
RUN cp bin/* /usr/libexec/cni
WORKDIR /

# network configs
RUN mkdir -p /etc/cni/net.d
RUN curl -qsSL -o 99-loopback.conf https://raw.githubusercontent.com/containers/libpod/master/cni/87-podman-bridge.conflist
RUN cp 99-loopback.conf /etc/cni/net.d/99-loopback.conf
WORKDIR /

# registries and policies
RUN mkdir -p /etc/containers
RUN curl https://raw.githubusercontent.com/projectatomic/registries/master/registries.fedora -o /etc/containers/registries.conf
RUN curl https://raw.githubusercontent.com/containers/skopeo/master/default-policy.json -o /etc/containers/policy.json
WORKDIR /

# build libpod
RUN git clone https://github.com/containers/libpod/ $GOPATH/src/github.com/containers/libpod
WORKDIR $GOPATH/src/github.com/containers/libpod
RUN make BUILDTAGS="selinux seccomp"
RUN make install PREFIX=
WORKDIR /
