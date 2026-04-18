Install dependencies:

    sudo apt update
    sudo apt install build-essential flex bison libssl-dev libelf-dev bc python3 pahole cpio

Download Kernel sources and checkout to the tag for the kernel in use:

    git clone --depth 1 -b linux-msft-wsl-$(uname -r|cut -d- -f1) https://github.com/microsoft/WSL2-Linux-Kernel.git
    cd WSL2-Linux-Kernel

Prepare and configure:

    mv .git .git_DISABLED # Get rid of "+" at the end (LOCALVERSION and LOCALVERSION_AUTO)
    make kernelrelease # Make sure there is no "+" at the end!

    cat /proc/config.gz | gunzip > .config
    make olddefconfig

Enable DM_CACHE:

    ./scripts/config --module CONFIG_DM_CACHE
    ./scripts/config --module CONFIG_DM_CACHE_SMQ
    ./scripts/config --module CONFIG_DM_WRITECACHE
    
Build:
    
    make -j$(nproc) prepare modules_prepare
    make -j$(nproc) KBUILD_MODPOST_WARN=1 M=drivers/md modules

Install

    sudo make M=drivers/md modules_install
    sudo depmod -a

Test

    modprobe dm-cache && lsmod | grep dm

Make it permanent -- does not work yet, since WSL resets the kernel directories on restart... work in progress...

    echo "dm-cache" | sudo tee /etc/modules-load.d/dm-cache.conf


