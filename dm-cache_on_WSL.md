
Install dependencies:

    sudo apt update
    sudo apt install build-essential flex bison libssl-dev libelf-dev bc python3 pahole cpio

Download Kernel sources and checkout to the tag for the kernel in use:

    cd $HOME
    git clone --depth 1 -b linux-msft-wsl-$(uname -r|cut -d- -f1) https://github.com/microsoft/WSL2-Linux-Kernel.git
    cd WSL2-Linux-Kernel

Prepare and configure:

    rm -rf $HOME/WSL2-Linux-Kernel/.git # Get rid of "+" at the end (LOCALVERSION and LOCALVERSION_AUTO)
    make kernelrelease # Make sure there is no "+" at the end!

    cat /proc/config.gz | gunzip > .config
    make olddefconfig


# Option A (as a module, restored at boot time)

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

### Make it permanent - self-contained systemd unit that re-installs the script at boot time

    sudo mkdir /modules
    sudo chown root:root /modules
    sudo chmod 0700 /modules
    sudo rm -rf /modules/WSL2-Linux-Kernel
    sudo mv $HOME/WSL2-Linux-Kernel /modules/
    
    sudo tee /etc/systemd/system/inject-kernel-modules.service <<'EOF'
    [Unit]
    Description=Inject Compiled Kernel Modules
    DefaultDependencies=no
    Before=local-fs-pre.target
    After=systemd-modules-load.service
    
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    WorkingDirectory=/modules/WSL2-Linux-Kernel/
    ExecStart=/usr/bin/make M=drivers/md modules_install
    ExecStart=/usr/sbin/depmod -a
    
    [Install]
    WantedBy=local-fs-pre.target
    EOF



    sudo tee /etc/systemd/system/activate-lvm.service <<'EOF'
    [Unit]
    Description=Activate LVM Volumes
    DefaultDependencies=no
    After=inject-kernel-modules.service
    Requires=inject-kernel-modules.service
    Before=local-fs-pre.target
    
    [Service]
    Type=oneshot
    ExecStart=/sbin/vgchange -ay
    RemainAfterExit=yes
    
    [Install]
    WantedBy=local-fs-pre.target
    EOF



    sudo systemctl daemon-reload
    sudo systemctl enable inject-kernel-modules.service activate-lvm.service


This survives a wsl --shutdown and dm-cache will automatically be loaded when required (by mount/fstab)


# Option B (compile as built-in and provide a custom kernel in wsl.config)

That's a tale for another day ;P

