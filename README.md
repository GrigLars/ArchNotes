# Installing Arch Linux and KDE on Virtualbox
## the Quick Generic Way

I made this file and some notes that should help somebody in the future.  The Arch Linux install notes are great, but they aren't perfect if you're in a hurry. Here are the commands you should run in roughly the correct order.  An explanation will be below.

    timedatectl set-ntp true
    cfdisk /dev/sda

Enter in what you want the disk to be partitioned as.  I just took two chunks: sda1 was bootable root, sda2 was swap.

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
    127.0.1.1	${HOSTNAME}.punkadyne ${HOSTNAME}" >> /etc/hosts
    grub-install --target=i386-pc /dev/sda && grub-mkconfig -o /boot/grub/grub.cfg
    systemctl enable sddm.service
    systemctl enable NetworkManager.service
    useradd -m joeuser && \
    echo 'joeuser ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/10_joeuser && \
    passwd joeuser
    
Hit `[control+D]` to exit chroot, and then type `reboot`

Log in as "joeuser" with the password you supplied.  The fiorst KDE session might take a bit with a lot of blank screen at first.  Next, make sure the guest additions are mounted.  

    sudo -i
    mount /dev/cdrom /mnt
    sh /mnt/VboxLinuxAdditions.run
    
Then reboot again. If you have issues with resizing screen resolution, I found shutting the instance down and selecting Settings > Display > VBoxVGA and disabling 3D accelleration fixed it. There may be other options, but I was not worried about it.
