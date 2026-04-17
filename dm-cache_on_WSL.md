Install dependencies:

    sudo apt update
    sudo apt install build-essential flex bison libssl-dev libelf-dev bc python3 pahole cpio

Download Kernel sources and checkout to the tag for the kernel in use:

    git clone --depth 1 -b linux-msft-wsl-$(uname -r|cut -d- -f1) https://github.com/microsoft/WSL2-Linux-Kernel.git

Prepare and configure:

    cd WSL2-Linux-Kernel
    cat /proc/config.gz | gunzip > .config
    touch .scmversion
    make olddefconfig

Enable DM_CACHE:

    ./scripts/config --module CONFIG_DM_CACHE
    ./scripts/config --module CONFIG_DM_CACHE_SMQ
    ./scripts/config --module CONFIG_DM_WRITECACHE
    
Build:
    
    ##make -j$(nproc) prepare
    make -j$(nproc) prepare modules_prepare
    ###make -j$(nproc) M=drivers/md modules
    make KBUILD_MODPOST_WARN=1 M=drivers/md modules

Install

    sudo make M=drivers/md modules_install
    sudo depmod -a

Test

    modprobe dm-cache && lsmod | grep dm

    
