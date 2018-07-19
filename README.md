# Unattended-ubuntu16.04-install
This repository will guide you on how to do a completely unattended install of ubuntu 16.04
## Mount the ISO File
You will need to mount the ISO files so that you can edit the pertinent files.
<pre><code>
mkdir -p /mnt/iso
mount -o loop ~/Downloads/ubuntu-16.04.1-desktop-amd64.iso /mnt/iso
</pre></code>
## Copy the Files
Now you need to copy the mounted files so that you can edit it. Do it by:
<pre><code>
mkdir -p ~/preseed_ubuntu
cp -rT /mnt/iso ~/preseed_ubuntu
</pre></code>
## Make the files editable 
We can do that by using `chmod`:
<pre><code>
sudo chmod -R ~/preseed_ubuntu
</pre></code>
## Edit the txt.cfg file
We have to add an option of Automated Install that will install everything automatically. You'll find the `txt.cfg` file in ` isolinux` or `syslinux` folder whichever is present. After that open the file with your favourite editor. 
<pre><code>
default autoinstall
label autoinstall
  menu label ^Autoinstall Ubuntu Server
  kernel /install/vmlinuz
  append debian-installer/locale=en_US debconf/priority=critical kbd-chooser/method=us locale=en_US console-setup/layoutcode=us file=/cdrom/preseed/ubuntu-server.seed initrd=/install/initrd.gz auto=true priority=high preseed/file=/cdrom/preseed/file.seed --
label install
  menu label ^Install Ubuntu Server
  kernel /install/vmlinuz
  append  file=/cdrom/preseed/ubuntu-server.seed vga=788 initrd=/install/initrd.gz quiet ---
label hwe-install
  menu label ^Install Ubuntu Server with the HWE kernel
  kernel /install/hwe-vmlinuz
  append  file=/cdrom/preseed/hwe-ubuntu-server.seed vga=788 initrd=/install/hwe-initrd.gz quiet ---
label maas
  menu label ^Install MAAS Region Controller
  kernel /install/vmlinuz
  append   modules=maas-region-udeb vga=788 initrd=/install/initrd.gz quiet ---

label maasrack
  menu label ^Install MAAS Rack Controller
  kernel /install/vmlinuz
  append   modules=maas-rack-udeb vga=788 initrd=/install/initrd.gz quiet ---
label check
  menu label ^Check disc for defects
  kernel /install/vmlinuz
  append   MENU=/bin/cdrom-checker-menu vga=788 initrd=/install/initrd.gz quiet ---
label memtest
  menu label Test ^memory
  kernel /install/mt86plus
label hd
  menu label ^Boot from first hard disk
  localboot 0x80
</pre></code>
Doing so will add one more menu entry but you will still have to hit Enter key for it to start.
## Automating the BOOT menu
If you want to prevent that step too, add a TOTALTIMEOUT of 10 in `isolinux.cfg` or `syslinux.cfg` and `prompt.cfg` in the same folder.
isolinux.cfg:
<pre><code>
# D-I config version 2.0
# search path for the c32 support libraries (libcom32, libutil etc.)
path 
include menu.cfg
default vesamenu.c32
prompt 0
timeout 10
ui gfxboot bootlogo
TOTALTIMEOUT 10
</pre></code>
prompt.cfg:
<pre><code>
prompt 0
display f1.txt
timeout 10
include menu.cfg
include exithelp.cfg

f1 f1.txt
f2 f2.txt
f3 f3.txt
f4 f4.txt
f5 f5.txt
f6 f6.txt
f7 f7.txt
f8 f8.txt
f9 f9.txt
f0 f10.txt
TOTALTIMEOUT 10
</pre></code>
## Editing and Adding the preseed file
Add file named `file.seed` in the `preseed` folder:
<pre><code>
# regional setting
d-i debian-installer/language                               string      en_US:en
d-i debian-installer/country                                string      US
d-i debian-installer/locale                                 string      en_US
d-i debian-installer/splash                                 boolean     false
d-i localechooser/supported-locales                         multiselect en_US.UTF-8
d-i pkgsel/install-language-support                         boolean     true

# keyboard selection
d-i console-setup/ask_detect                                boolean     false
d-i keyboard-configuration/modelcode                        string      pc105
d-i keyboard-configuration/layoutcode                       string      us
d-i keyboard-configuration/variantcode                      string      intl
d-i keyboard-configuration/xkb-keymap                       select      us(intl)
d-i debconf/language                                        string      en_US:en

# network settings
d-i netcfg/choose_interface                                 select      auto
d-i netcfg/dhcp_timeout                                     string      5
d-i netcfg/get_hostname                                     string      ubuntu
d-i netcfg/get_domain                                       string      ubuntu

# mirror settings
d-i mirror/country                                          string      manual
d-i mirror/http/hostname                                    string      archive.ubuntu.com
d-i mirror/http/directory                                   string      /ubuntu
d-i mirror/http/proxy                                       string

# clock and timezone settings
d-i time/zone                                               string      Asia/Kolkata
d-i clock-setup/utc                                         boolean     false
d-i clock-setup/ntp                                         boolean     true

# user account setup
d-i passwd/root-login                                       boolean     false
d-i passwd/make-user                                        boolean     true
d-i passwd/user-fullname                                    string      USER
d-i passwd/username                                         string      ubuntu
d-i passwd/user-password                                    password    asdf
d-i passwd/user-password-again                              password    asdf
d-i user-setup/allow-password-weak                          boolean     true
d-i passwd/user-default-groups                              string      adm cdrom dialout lpadmin plugdev sambashare
d-i user-setup/encrypt-home                                 boolean     false

# configure apt
d-i apt-setup/restricted                                    boolean     true
d-i apt-setup/universe                                      boolean     true
d-i apt-setup/backports                                     boolean     true
d-i apt-setup/services-select                               multiselect security
d-i apt-setup/security_host                                 string      security.ubuntu.com
d-i apt-setup/security_path                                 string      /ubuntu
tasksel tasksel/first                                       multiselect standard system utilities, Basic Ubuntu server, OpenSSH server
d-i pkgsel/upgrade                                          select      safe-upgrade
d-i pkgsel/update-policy                                    select      none
d-i pkgsel/updatedb                                         boolean     true

# disk partitioning
d-i partman/confirm_write_new_label                         boolean     true
d-i partman/choose_partition                                select      finish
d-i partman/confirm_nooverwrite                             boolean     true
d-i partman/confirm                                         boolean     true
d-i partman-auto/purge_lvm_from_device                      boolean     true
d-i partman-lvm/device_remove_lvm                           boolean     true
d-i partman-lvm/confirm                                     boolean     true
d-i partman-lvm/confirm_nooverwrite                         boolean     true
d-i partman-auto-lvm/no_boot                                boolean     true
d-i partman-md/device_remove_md                             boolean     true
d-i partman-md/confirm                                      boolean     true
d-i partman-md/confirm_nooverwrite                          boolean     true
d-i partman-auto/method                                     string      lvm
d-i partman-auto-lvm/guided_size                            string      max
d-i partman-partitioning/confirm_write_new_label            boolean     true

# grub boot loader
d-i grub-installer/only_debian                              boolean     true
d-i grub-installer/with_other_os                            boolean     true

# Software selection
d-i pkgsel/include                                          string      openssh-server

# finish installation
d-i finish-install/reboot_in_progress                       note
d-i finish-install/keep-consoles                            boolean     false
d-i cdrom-detect/eject                                      boolean     true
#  to switch the machine off after install uncomment these
#d-i debian-installer/exit/halt                              boolean     false
#d-i debian-installer/exit/poweroff                          boolean     true

# setup firstrun script. (these scripts will run just after installation, uncomment if you want to use)
#d-i preseed/late_command                                    string 		 
</pre></code>
To view the official ubuntu example [refer here](https://help.ubuntu.com/lts/installation-guide/example-preseed.txt).
 As the file is self explanatory, you can easily understand and change the parameters.
## Create new iso file
run the following code as `sudo` while being in the directory:
<pre><code>
mkisofs -D -r -V "UNATTENDED_UBUNTU" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ~/ubuntu16-unattended-install.iso ~/preseed_ubuntu
</pre></code>
## Burning the iso file
Congratulations, You're done.
Now, test the iso file on a virtual machine.

#### Bug or suggestion?
Feel free to report any problem :)

