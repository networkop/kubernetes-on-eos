# Building EOS-compatible kubernetes binaries

EOS is running on a 64-bit Fedora with 32-bit userspace libraries. This document describes how to build all the necessary 32-bit binaries.

It's possible to build the required binaries in two ways:

1. Directly on EOS - for example using vEOS image by bind-mounting the `/usr` and `/bin` directories onto a separate disk with extra space. This process is described [here](https://github.com/choco-loo/arista-extensions)
2. On any 32-bit Fedora 18 distribution

This document will describe the latter process. The following version of packages will be built:

```
export K8S_VERSION="v1.12.4"
export ETCD_VERSION="v3.3.10"
export FLANNEL_VERSION="v0.10.0"
```

All final binaries will be stored in `$HOME/bin`

```
mkdir $HOME/bin
```

# Prerequisites

Download the 32-bit Fedora 18 ISO image.

```
$ ls *.iso
Fedora-18-i386-DVD.iso
```

Use this image to build a VM with 20GB disk space and roughly 1vCPU and 4GB of RAM. Login that VM and make sure you've got internet access. All subsequent steps are done from inside that VM

# Installing dependencies


## Install tar, wget, gcc and git

```
yum install wget tar git gcc glibc-static -y
```

## Install golang

```
wget https://dl.google.com/go/go1.11.4.linux-386.tar.gz
tar -C /usr/local -xzf go1.11.4.linux-386.tar.gz
export PATH=$PATH:/usr/local/go/bin
mkdir $HOME/go
export GOPATH=$HOME/go
```

# Build kubernetes binaries

```
git clone --depth 1 --branch $K8S_VERSION git://github.com/kubernetes/kubernetes.git && cd kubernetes
make WHAT='cmd/hyperkube cmd/kubectl'
mv _output/local/bin/linux/386/hyperkube $HOME/bin/
mv _output/local/bin/linux/386/kubectl $HOME/bin/
```

# Build etcd binaries

```
git clone --depth 1 --branch $ETCD_VERSION git://github.com/etcd-io/etcd.git && cd etcd
./build
mv bin/* $HOME/bin/
```

# Build flannel binaries

```
mkdir -p $HOME/go/github.com/coreos
git clone --depth 1 --branch $FLANNEL_VERSION git://github.com/coreos/flannel.git \                                               $HOME/go/src/github.com/coreos/flannel \
          && cd $HOME/go/src/github.com/coreos/flannel
CGO_ENABLED=1 make dist/flanneld
mv dist/flanneld $HOME/bin 
```

# (Plan-B) Cross-platform build (need to clean up later)

```
git clone --depth 1 --branch v1.12.4 git://github.com/kubernetes/kubernetes.git && cd kubernetes
git apply ../git.diff
KUBE_BUILD_PLATFORMS="linux/386" CGO_ENABLED=0 make WHAT='cmd/hyperkube cmd/kubectl'

```



