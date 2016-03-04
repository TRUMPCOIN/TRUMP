
--Updated 4/5/2014 by TrumpCoinDevs--
--fixed QT source Links--
--Updated to TrumpCoin Folders--
--updated links to boost 1_55_0
--Added FAQ--



<img src="http://209.208.111.8/splash.png" alt="TrumpCoin">


Gavin's notes on getting gitian builds up and running using KVM:

These instructions distilled from:
  https://help.ubuntu.com/community/KVM/Installation
... see there for complete details.

You need the right hardware: you need a 64-bit-capable CPU with hardware virtualization support (Intel VT-x or AMD-V). Not all modern CPUs support hardware virtualization.

You probably need to enable hardware virtualization in your machine's BIOS.

You need to be running a recent version of 64-bit-Ubuntu, and you need to install several prerequisites:
  sudo apt-get install ruby apache2 git apt-cacher-ng python-vm-builder qemu-kvm

Sanity checks:
  sudo service apt-cacher-ng status   # Should return apt-cacher-ng is running
  ls -l /dev/kvm   # Should show a /dev/kvm device


Once you've got the right hardware and software:

    git clone git://github.com/IParn/TrumpCoin.git
    git clone git://github.com/devrandom/gitian-builder.git
    mkdir gitian-builder/inputs
    cd gitian-builder/inputs
    # Inputs for Linux and Win32:
    wget -O miniupnpc-1.6.tar.gz 'http://miniupnp.tuxfamily.org/files/download.php?file=miniupnpc-1.6.tar.gz'
    wget 'http://fukuchi.org/works/qrencode/qrencode-3.2.0.tar.bz2'
    # Inputs for Win32: (Linux has packages for these)
    wget 'http://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2'
    wget 'https://svn.boost.org/trac/boost/raw-attachment/ticket/7262/boost-mingw.patch'
    wget 'https://www.openssl.org/source/openssl-1.0.1g.tar.gz'
    wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
    wget 'https://downloads.sourceforge.net/project/libpng/zlib/1.2.6/zlib-1.2.6.tar.gz'
    wget 'https://downloads.sourceforge.net/project/libpng/libpng15/older-releases/1.5.9/libpng-1.5.9.tar.gz'
    wget 'http://ftp.lfs-matrix.net/pub/blfs/conglomeration/qt4/qt-everywhere-opensource-src-4.8.3.tar.gz'
    cd ../..

    cd gitian-builder
    sudo bin/make-base-vm --arch i386
    sudo bin/make-base-vm --arch amd64 
    sudo bin/make-base-vm --arch amd64 --suite precise
    cd ..

    # Build Linux release:
    cd TrumpCoin
    git pull
    cd ../gitian-builder
    git pull
    sudo ./bin/gbuild --commit TrumpCoin=HEAD ../TrumpCoin/contrib/gitian-descriptors/gitian.yml

    # Build Win32 dependencies: (only needs to be done once, or when dependency versions change)
    sudo ./bin/gbuild --commit TrumpCoin=HEAD ../TrumpCoin/contrib/gitian-descriptors/boost-win32.yml
    sudo ./bin/gbuild --commit TrumpCoin=HEAD ../TrumpCoin/contrib/gitian-descriptors/deps-win32.yml
    sudo ./bin/gbuild --commit TrumpCoin=HEAD ../TrumpCoin/contrib/gitian-descriptors/qt-win32.yml

    # Build Win32 release:
    sudo ./bin/gbuild --commit TrumpCoin=HEAD ../TrumpCoin/contrib/gitian-descriptors/gitian-win32.yml

---------------------

gitian-builder now also supports building using LXC. See
  https://help.ubuntu.com/12.04/serverguide/lxc.html
... for how to get LXC up and running under Ubuntu.

If your main machine is a 64-bit Mac or PC with a few gigabytes of memory
and at least 10 gigabytes of free disk space, you can gitian-build using
LXC running inside a virtual machine.

Here's a description of Gavin's setup on OSX 10.6:

1. Download and install VirtualBox from https://www.virtualbox.org/

2. Download the 64-bit Ubuntu Desktop 12.04 LTS .iso CD image from
  http://www.ubuntu.com/

3. Run VirtualBox and create a new virtual machine, using the
  Ubuntu .iso (see the VirtualBox documentation for details).
  Create it with at least 2 gigabytes of memory and a disk
  that is at least 20 gigabytes big.

4. Inside the running Ubuntu desktop, install:
  sudo apt-get install debootstrap lxc ruby apache2 git apt-cacher-ng python-vm-builder

5. Still inside Ubuntu, tell gitian-builder to use LXC, then follow the "Once you've got the right
  hardware and software" instructions above:
  export USE_LXC=1
  git clone git://github.com/bitcoin/bitcoin.git
  ... etc



FAQ:


ssh: connect to host localhost port 2223: Connection refused
This means that qemu has failed to start. Try editing libexec/start-target and changing "kvm" to "qemu".
 If that fails, look in var/target.log for any error messages.
Another reason may be that you are using a VPS , try installing Ubuntu and gitian in a Desktop Computer.(Works for me)


Connection timed out during banner exchange
This means your qemu VM didn't manage to boot far enough for ssh to be running. 
There appears to be a lurking bug in some python-vm-builder versions that causes bootloader installation to fail.


cp: cannot open `base-lucid-i386.qcow2' for reading: Permission denied
Code:
sudo chmod 644 base-lucid-i386.qcow2



Updating apt-get repository (log in var/install.log)
root@localhost's password:
ubuntu@localhost's password:

Delete all your VMs, and reinstall them Clean. 
( I had this problem as well when I was compiling using gitian. I'm confident theres a better solution but restarting the 
vm instance that I was running kvm in solved the issue. Likely something to do with residuals left from the previous 
dependency builds that werent removed correctly wjem they completed. 

I stopped digging after the reboot solved the issue.)



If you need more help please visit TrumpCoin Forum at www.trumpcoin.rocks


