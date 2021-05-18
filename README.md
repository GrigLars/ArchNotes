# Installing Arch Linux and KDE on Virtualbox 
I made this file and some notes that should help somebody in the future.  The Arch Linux install notes are great, but they aren't perfect if you're in a hurry. Here are the commands you should run in roughly the correct order.  An explanation will be below.  This assumed you got the install disk booted, or used iPXE, and are now ready to start the install process.  I would say this requires medium-advanced experience, and I know that this may make some Arch users mad because I skip a lot of things, but I just wanted to get Arch installed to test some things.

Note: Arch documentation is thurough and amazing.  Kudos to the people who spend their time doing this. 

## The Quick Generic Way
These are kind of specific to my preferences and environment, so watch out for the cutting and pasting, okay?

    timedatectl set-ntp true
    cfdisk /dev/sda

Enter in what you want the disk to be partitioned as.  I just took two chunks: sda1 was bootable root, sda2 was swap, and so the rest of these steps reflect that.  You shouldn't be cutting and pasting blindly, any way.  Right?  RIGHT??

    mkfs.ext4 /dev/sda1
    mkswap /dev/sda2
    swapon
    mount /dev/sda1 /mnt
    pacstrap /mnt base linux make grub gcc perl dhclient vim xorg plasma plasma-wayland-session kde-applications linux-headers sudo terminator 
    genfstab -U /mnt >> /mnt/etc/fstab
    arch-chroot /mnt
    ln -sf /usr/share/zoneinfo/UTC /etc/localtime
    hwclock --systohc
    locale-gen
    sed -ie 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    HOSTNAME=vm-archlinux
    echo "${HOSTNAME}" > /etc/hostname
    echo "
    127.0.0.1	localhost
    ::1 		localhost
    127.0.1.1	${HOSTNAME}.mydomain ${HOSTNAME}" >> /etc/hosts
    grub-install --target=i386-pc /dev/sda && grub-mkconfig -o /boot/grub/grub.cfg
    systemctl enable sddm.service
    systemctl enable NetworkManager.service
    useradd -m joeuser && \
    echo 'joeuser ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/10_joeuser && \
    passwd joeuser
    
Hit `[control+D]` to exit chroot, and then type `reboot`

Log in as "joeuser" with the password you supplied.  The first KDE session might take a bit with a lot of blank screen at first, just be patient.  Next, make sure the guest additions are mounted.  

    sudo -i
    mount /dev/cdrom /mnt
    sh /mnt/VboxLinuxAdditions.run
    
Then reboot again. If you have issues with resizing screen resolution, I found shutting the instance down and selecting Settings > Display > VBoxVGA and disabling 3D accelleration fixed it. There may be other options, but I was not worried about it.

## More Detailed Explanations

    timedatectl set-ntp true
This ensures the system clock is accurate, which is always a good idea.

    cfdisk /dev/sda
The cfdisk (which I think stands for "Curses fdisk" where "curses" means text mode that allows terminal drawings) is much easier to use than plain old fdisk, IMHO. In this case, it's the VirtualBox disk, which will show up as `/dev/sda` or "the first SCSI/SATA disk" (the second would be sdb, then sdc, and so on).

    mkfs.ext4 /dev/sda1
    mkswap /dev/sda2
    swapon
    mount /dev/sda1 /mnt
Format the partitions.  In this case, ext4. Format swap and turn it on.  Then mount the root directory so we can read-write to it.

    pacstrap /mnt base linux make grub gcc perl dhclient vim xorg plasma plasma-wayland-session kde-applications linux-headers sudo terminator
This installs what we need for a working Arch Linux system with KDE on the next boot.  
**base**: base linux programs and files, 
**linux**: the linux kernel (not needed for containers like docker, LXC, etc), 
**make**: stuff to interpret makefiles, needed for VirtualBox guest additions, 
**grub**: needed to boot the system, 
**gcc**: the C compiler, needed for VirtualBox guest additions, 
**perl**: perl interpreter, needed for VirtualBox guest additions, 
**dhclient**: get your IP by dhcp, 
**vim**: my favorite text editor (others are nano, emacs, or joe), 
**xorg**: the Xorg display server, 
**plasma**: KDE's rendering engine, 
**plasma-wayland-session**: KDE's wayland (compositor protocol) session, 
**kde-applications**: this is the KDE meta-package, it's like "kde-full" on other systems, 
**linux-headers**: the kernel headers and configs, needed for VirtualBox guest additions, 
**sudo**: the "super user do" package for running root commands, 
**terminator**: my favorite terminal program like konsole or xterm.

    genfstab -U /mnt >> /mnt/etc/fstab
Generates the file system table (what mounts where)
    
    arch-chroot /mnt
The "change root to" command.  This is the core of containers like docker, by the way. This changes the metal state of the OS to think that what you told it root is, is root.  So if you have `/mnt/etc/foo.conf` but then do `arch-chroot /mnt` then it will think the file is in `/etc/foo.conf`. That's a really over-simplified way to think of it.  But it's needed to make shortcuts and use the correct linked libraries.

    ln -sf /usr/share/zoneinfo/UTC /etc/localtime
Links the timezone.  I always use UTC because that's what the industry uses.  You can see others (like EST, EDT5EST, Pacific, Asia/Japan, etc.) by doing `ls /usr/share/zoneinfo/` and see what's in there)

    hwclock --systohc
Sync the hardware clock to a file 

    locale-gen
Generate locales

    sed -ie 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
Uncomment English, US Unicode Transformation Format for 8-bit, and then add it to the locale.conf

    export HOSTNAME=vm-archlinux
    echo "${HOSTNAME}" > /etc/hostname
    echo "
    127.0.0.1	localhost
    ::1 		localhost
    127.0.1.1	${HOSTNAME}.mydomain ${HOSTNAME}" >> /etc/hosts
This sets the hostname to what you want, I chose "vm-archlinux."  Then make sure the hostname is recorded through a reboot, and put it in the hosts file.

    grub-install --target=i386-pc /dev/sda && grub-mkconfig -o /boot/grub/grub.cfg
Install the GRand Unified Boot loader for Intel-based systems onto the first disk (not parition, disk), then make the grub config for use at boot time.

    systemctl enable sddm.service
    systemctl enable NetworkManager.service
This enabled two services: Simple Desktop Display Manager (the login window) and NetworkManager which KDE uses to manage your ethernet connection.

    useradd -m joeuser && \
    echo 'joeuser ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/10_joeuser && \
    passwd joeuser
Add a user with admin capabilities.  Since we don't set the root password in this document, this is an essential step.

### Links:
https://wiki.archlinux.org/title/Installation_guide
https://itsfoss.com/install-kde-arch-linux/
https://wiki.archlinux.org/title/GRUB
https://wiki.archlinux.org/title/Network_configuration
