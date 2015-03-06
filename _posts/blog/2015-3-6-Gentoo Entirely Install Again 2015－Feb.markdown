---
layout: post
title:  "Gentoo Entirely Install Again 2015－Feb "
date:   2015-3-6
author: me
categories: 
- blog
- Linux流
thumb: thumb06.png
---

<img src="http://liubai.info/assets/img/work/6.jpg" style="width:500px;height=248px">


<!--more-->


ubuntu live cd 进入 ternimal

    $sudo su
    $nautilus gparted
# 挂载 #    
格式化 sda7 sda8
建立/mnt/gentoo

    $mount /dev/sda8 /mnt/gentoo
    $mkdir /mnt/gentoo/boot
    $mount /dev/sda7 /mnt/gentoo/boot
    $cd /mnt/gentoo 
    links http://www.gentoo.org/main/en/mirrors.xml

选镜像，在学校选择厦门大学的镜像，否则网易或者搜狐。
下载releases/x86/autobuilds/20150303/stage3-i686-20150303.tar.bz2   

    $time tar xjpf stage3*
    $cd /mnt/gentoo/usr

下载releases/snapshots/current/portage-20150224.tar.bz2

    $time tar xjf portage*

# chroot #
    
    livecd usr # cd /
    livecd / # mount -t proc proc /mnt/gentoo/proc
    livecd / # mount -o bind /dev /mnt/gentoo/dev
    livecd / # cp -L /etc/resolv.conf /mnt/gentoo/etc/
    livecd / # chroot /mnt/gentoo /bin/bash
    livecd / # env-update && source /etc/profile
    >>> Regenerating /etc/ld.so.cache...
    livecd / # cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    livecd / # cd /etc
    livecd etc # echo "127.0.0.1 McGrady.at.myplace McGrady localhost" > hosts
    livecd etc # sed -i -e 's/HOSTNAME.*/HOSTNAME="McGrady"/' conf.d/hostname
    livecd etc # hostname McGrady
    livecd etc # hostname -f
    McGrady.at.myplace
    livecd etc # time emerge gentoo-sources
    
    livecd etc # cd /usr/src/linux
    livecd linux # make menuconfig

# 内核配置 #


<img src="http://liubai.qiniudn.com/kernel.png" style="width:500px;height=248px">

    Gentoo Linux  --->
    [ ] 64-bit kernel 
    Processor type and features  --->
        High Memory Support (64GB)  --->
    Device Drivers  --->
        [*] Network device support  ---> 
            [*]   Ethernet driver support  ---> 
                [*]   Atheros devices
                    < >     Atheros L2 Fast Ethernet support
                    < >     Atheros/Attansic L1 Gigabit Ethernet support
                    < >     Atheros L1E Gigabit Ethernet support
                    < >     Atheros L1C Gigabit Ethernet support 
                    <*>     Qualcomm Atheros AR816x/AR817x support 

            <*>   PPP (point-to-point protocol) support
                <*>     PPP over Ethernet
                <*>     PPP support for async serial ports
                <*>     PPP support for sync tty ports
            [*]   Wireless LAN  --->
                <*>   Atheros Wireless Cards  ---> 
                    <*>   Atheros 802.11n wireless cards support
                        [*]     Atheros ath9k PCI/PCIe bus support
        <*> Sound card support  --->
            <*>   Advanced Linux Sound Architecture  ---> 
                [*]   PCI sound devices  --->
                    <*>   Intel/SiS/nVidia/AMD/ALi AC97 Controller
                    <*>   Intel/SiS/nVidia/AMD MC97 Modem
                    <*>   Asus Virtuoso 66/100/200 (Xonar)
    File systems  --->
        <*> FUSE (Filesystem in Userspace) support
            DOS/FAT/NT Filesystems  --->
                <*> NTFS file system support
                    [*]   NTFS write suppor    


  



livecd linux # time make -j5
livecd linux # make modules_install
livecd linux # cp arch/i386/boot/bzImage /boot/kernel
emerge genkernel
genkernel --install --no-ramdisk-modules initramfs




livecd linux # cd /etc
livecd etc # nano -w fstab

<img src="http://liubai.qiniudn.com/fstab.png" style="width:500px;height=248px">

    /dev/sda7   /boot   ext4noauto,noatime  1 2
    /dev/sda8   /   ext4noatime 0 1
    /dev/cdrom  /mnt/cdrom  autonoauto,ro   0 0
    /dev/fd0/mnt/floppy autonoauto  0 0
    /dev/sda9   /home/tracy/mount/tracy ext4noatime 0 1
    /dev/sda10  /home/tracy/mount/arch  ext4noatime 0 1
    /dev/sda1   /home/tracy/mount/win   ntfs-3g noatime 0 1
    /dev/sda5   /home/tracy/mount/media ntfs-3g noatime 0 1
    /dev/sda6   /home/tracy/mount/geek  ntfs-3g noatime 0 1

# 网络root密码守护进程 #

    livecd etc # cd init.d
    livecd init.d # ln -s net.lo net.enp4s0f2
    livecd init.d # cd ../conf.d
    livecd conf.d # echo 'config_enp4s0f2="192.168.1.10 netmask 255.255.255.0 brd 192.168.1.255"' >net
    livecd conf.d # echo 'routes_enp4s0f2="default via 192.168.1.1"' >net
    livecd conf.d # rc-update add net.enp4s0f2 default
    livecd conf.d # passwd
    New UNIX password: type_the_password
    Retype new UNIX password: type_the_password_again
    passwd: password updated successfully
    livecd conf.d # time emerge syslog-ng vixie-cron
    livecd conf.d # rc-update add syslog-ng default
    livecd conf.d # rc-update add vixie-cron default

$emerge ppp rp-pppoe
$emerge --ask sys-boot/grub:0
这个是grub-0.97  

nano -w /boot/grub/grub.conf



    default 0
    timeout 30
    splashimage=(hd0,6)/boot/grub/sp.xpm.gz
    
    title 06
    root (hd0,6)
    kernel /boot/kernel-3.18.7 root=/dev/ram0 real_root=/dev/sda8
    initrd /boot/initramfs-genkernel-x86-3.17.8-gentoo-r1
    
    title 09
    root (hd0,9)
    kernel /boot/vmlinuz-linux root=/dev/sda10 ro
    initrd /boot/initramfs-linux.img
    
    title 00
    root (hd0,0)
    makeactive
    chainloader +1

写完配置文件安装grub:0.97

    livecd conf.d # grub
    Probing devices to guess BIOS drives. This may take a long time.
    
    grub> root (hd0,6)
     Filesystem type is ext2fs, partition type 0xfd
    
    grub> setup (hd0)
     Checking if "/boot/grub/stage1" exists... yes
     Checking if "/boot/grub/stage2" exists... yes
     Checking if "/boot/grub/e2fs_stage1_5" exists... yes
     Running "embed /boot/grub/e2fs_stage1_5 (hd0)"...  16 sectors are embedded.
    succeeded
     Running "install /boot/grub/stage1 (hd0) (hd0)1+16 p (hd0,0)/boot/grub/stage2 /boot/
    grub/menu.lst"... succeeded
    Done.
    
    grub> quit

mybox ~ # useradd -g users -G lp,wheel,audio,cdrom,portage,cron -m tracy
mybox ~ # passwd tracy


mybox ~ # emerge mirrorselect
mybox ~ # mirrorselect -i -o >> /etc/portage/make.conf
mybox ~ # mirrorselect -i -r -o >> /etc/portage/make.conf
(Usually, (the number of processors + 1) is a good value)
mybox ~ # echo 'MAKEOPTS="-j5"' >> /etc/portage/make.conf
mybox # nano -w /etc/locale.gen

    en_US ISO-8859-1
    en_US.UTF-8 UTF-8
    zh_CN GB2312
    zh_CN.GBK GBK
    zh_CN.GB18030 GB18030
    zh_CN.UTF-8 UTF-8
    zh_HK BIG5-HKSCS
    zh_HK.UTF-8 UTF-8
    zh_TW BIG5
    zh_TW.UTF-8 UTF-8
    zh_CN GB18030

mybox # locale-gen
安装xorg驱动
root #emerge --ask --verbose --pretend x11-base/xorg-drivers
root #echo "x11-base/xorg-server udev" >> /etc/portage/package.use
root #emerge --ask x11-base/xorg-server
root #env-update
root #source /etc/profile
此时就把intel驱动安装上了  xf86-video-intel mesa 等等
cp /etc/X11/xinit/xinitrc /home/tracy/.xinitrc
nano ~/.xinitrc
exec ck-launch-session dbus-launch --sh-syntax --exit-with-session awesome


user $mkdir -p ~/.config/awesome
user $cp /etc/xdg/awesome/rc.lua ~/.config/awesome/rc.lua
user $emerge -a terminator

user $nano .config/terminator/config

<img src="http://liubai.qiniudn.com/terminator.png" style="width:500px;height=248px">


user $emerge -a app-i18n/scim-tables app-i18n/scim app-i18n/scim-pinyin

user $nano .xinitrc 

    export XMODIFIERS="@im=SCIM"
    export GTK_IM_MODULE=scim
    export QT_IM_MODULE=scim
    export XIM_PROGRAM=SCIM
    export XIM=SCIM
    exec awesome
    



livecd / # umount /mnt/gentoo/dev /mnt/gentoo/proc /mnt/gentoo/boot /mnt/gentoo








## gtk theme switch ##

选择gtk icon主题  配置.gtkrc-2.0

<img src="http://liubai.qiniudn.com/gtktheme.png" style="width:500px;height=248px">


## awsetbf feh ##

awsetbg是显示壁纸的命令  
wiki说需要feh的依赖 但是不安装也没有问题，但是一旦设置其他壁纸路径就会报错 参数也没有用 
没办法用了feh 
theme.wallpaper_cmd = { "feh --bg-scale /home/tracy/wallpaper/1.png" }

## make.conf ##

    GENTOO_MIRRORS="http://mirrors.163.com/gentoo/ http://mirrors.sohu.com/gentoo/ http://mirrors.stuhome.net/gentoo/ ftp://mirrors.stuhome.net/gentoo/ ftp://mirrors.xmu.edu.cn/gentoo http://mirrors.xmu.edu.cn/gentoo rsync://mirrors.xmu.edu.cn/gentoo/"
    
    SYNC="rsync://rsync1.cn.gentoo.org/gentoo-portage"
    MAKEOPTS="-j4"
    USE="wma wav nptl nptlonly -ipv6 -fortran unicode aac alsa cairo encode flac jpeg png ffmpeg gstreamer svg lua pdf xft X mp3 mp4 wifi wma truetype -kde -gnome -minimal -qt4 dbus session -udev startup-notification"
    VIDEO_CARDS="intel i915"



## vlc qt4 ##

<img src="http://liubai.qiniudn.com/vlc.png" style="width:500px;height=248px">

vlc安装 USE="X alsa avcodec avformat dbus dvbpsi encode ffmpeg flac gcrypt libav lua mp3 ncurses png postproc qt4 svg swscale truetype xcb skins"
忘记qt4 就没有界面
没有xml skins 标记不能更换皮肤

 
## nfts3g fuse ##

差点又忘记了ntfs3g  

<img src="http://liubai.qiniudn.com/ntfs.jpg" style="width:500px;height=248px">


## scim 小狼嚎输入法 ##

很久听说了这个输入法，据说乃是一个神器，不属于搜狗的，当初搜狗作为搜狐工具类应用的看家之作可谓是赞誉有加，用户体验相当不错，但是逐渐广告的盛行也是让人很是烦恼，加上不知道为什么，越来越不智能，只好试试这个，还不错的是比较fcitx可以方向键选词，在sougou养地习惯还是得以延续下去，例外拼音方面虽然有些bug，但是不大影响使用的，打字的效率还是很高的。
说一下安装：
1.系统的LANG locale设置

2.安装并且在bashrc和xinitrc加入 export 语句即可


 










