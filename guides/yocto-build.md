# Yocto build guide

---
system requirement reference: [link](https://docs.yoctoproject.org/scarthgap/ref-manual/system-requirements.html?utm_source=chatgpt.com)
### Build environments
- OS: Ubuntu 22.04
- At least 8 GB RAM
- At least 90 GB Disk space
### Install dependency package
```
sudo apt install -y \
    build-essential \
    chrpath \
    cpio \
    debianutils \
    diffstat \
    file \
    gawk \
    gcc \
    git \
    iputils-ping \
    libacl1 \
    liblz4-tool \
    locales \
    python3 \
    python3-git \
    python3-jinja2 \
    python3-pexpect \
    python3-pip \
    python3-subunit \
    socat \
    texinfo \
    unzip \
    wget \
    xz-utils \
    zstd \
    crul
```

### Configure locale
```
    sudo locale-gen en_US.UTF-8
```
```
    locale
```
### Install repo
```
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo \
    -o ~/bin/repo
```
```
chmod a+x ~/bin/repo
```
```
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
```
```
source ~/.bashrc
```
```
repo --version
```
### Create workspace
```
mkdir -p ~/yocto-zcu104
cd ~/yocto-zcu104
```

### Pull yocto repo
```
repo init -u https://github.com/Xilinx/yocto-manifests.git \
          -b rel-v2026.1 \
          -m default-edf.xml
```
```
repo sync
```
### Activate
```
source ./edf-init-build-env
```
- now path change to ~/yocto-zcu104/build
### choose machine in conf
```
nano ~/yocto-zcu104/build/conf/local.conf
```
at top of file you will see comment ```#MACHINE = "zynqmp-zcu104-sdt-full"```
so uncomment it and comment ```MACHINE ??= "qemuarm64"``` instead.
save and exit

### Test parsing
```
    bitbake -p
```
- If complete, no error do next step

### Build yocto!
```
    bitbake core-image-minimal
```
- If complete, no error do next step
### check file 
```
ls tmp/deploy/images/zynqmp-zcu104-sdt-full/ | grep .qemu-sd
```
- you will see file like ```core-image-minimal-zynqmp-zcu104-sdt-full.rootfs-[xxxxxxxxxxxxxx].wic.qemu-sd```
use this file to write on SD card in next step

### write to SD Card
- first check your target disk name 
```
lsblk
```
- unmount
```
sudo umount /dev/[xxx]*
```
- write to SD Card
```
sudo dd if=tmp/deploy/images/zynqmp-zcu104-sdt-full/core-image-minimal-zynqmp-zcu104-sdt-full.rootfs-[xxxxxxxxxxxxxx].wic.qemu-sd \
         of=/dev/sdd \
         bs=4M \
         status=progress \
         conv=fsync
```
```
sync
```
### !complete, go to zcu104 and boot it