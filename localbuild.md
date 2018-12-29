
https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/18/Everything/i386/os/

service docker start

cat > /etc/yum.repos.d/fedora.repo <<EOF
[fedora]
name=Fedora 18 - i686
failovermethod=priority
baseurl=https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/18/Everything/i386/os/
exclude=kernel,fedora-logos
exclude=kernel,fedora-logos
enabled=1
gpgcheck=0
diskspacecheck=0
EOF



bash sudo su
fdisk /dev/sdb
mkfs.ext2 /dev/sdb1

mkdir -p /mnt/sdb1
mount /dev/sdb1 /mnt/sdb1
mkdir -p /mnt/sdb1/{bin,var/cache/yum,usr}
rsync -axpHSD --numeric-ids -vP --delete /var/cache/yum/ /mnt/sdb1/var/cache/yum/
rsync -axpHSD --numeric-ids -vP --delete /usr/ /mnt/sdb1/usr/
rsync -axpHSD --numeric-ids -vP --delete /bin/ /mnt/sdb1/bin/

mount --bind /mnt/sdb1/var/cache/yum /var/cache/yum
mount --bind /mnt/sdb1/usr /usr
mount --bind /mnt/sdb1/bin /bin

yum install yum-utils
yumdownloader --resolve --destdir=/mnt/sdb1/downloads/git git
cd /mnt/sdb1/downloads/git
rpm2cpio ./git-1.8.0.1-1.fc18.i686.rpm | cpio -idmv
export PATH=$PATH:/mnt/sdb1/downloads/git/usr/bin/

cd ../
wget https://dl.google.com/go/go1.11.4.linux-386.tar.gz
tar -C /usr/local -xzf go1.11.4.linux-386.tar.gz
export PATH=$PATH:/usr/local/go/bin
mkdir /mnt/sdb1/go
export GOPATH=/mnt/sdb1/go

git clone  --branch v1.10.1 --depth 1 git://github.com/kubernetes/kubernetes.git k8s && cd k8s
mkdir _output
cp api/api-rules/violation_exceptions.list _output/violations.report
GO_ENABLED=1 make WHAT='cmd/kube-proxy'
scp _output/bin/kube-proxy admin@172.20.0.2:/mnt/flash/k8s/

./build/run.sh  make kubelet KUBE_BUILD_PLATFORMS=linux/386
