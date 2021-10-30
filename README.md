# **Arch Linux Notes to Self**

## Install Process

Following the [Installation Guide](https://wiki.archlinux.org/index.php/installation_guide).

`ip link`, `wifi-menu`, `iwctl` to get connected to the internet (if no ethernet)

### `iwctl`

Newer versions of the installation guide don't come with wifi-menu, relying on iwctl. Use the following commands in the iwctl prompt:

```
device list
station DEVICE scan
station DEVICE get-networks
station DEVICE connect SSID
```

`loadkeys uk`

`timedatectl set-ntp true`, `timedatectl status` (this seems to fuck up Windows clock, remember to reset it)

### Partitions, Filesystems, Mounting

`lsblk` to sanity check, `-f` option to see filesystems

`fdisk /dev/sdX` to select a disk

`n` to create a new partition, `p` to print, `d` to delete, `w` to write, `t` to change type

want a `boot` of about 512MiB and the rest can be `root` (set up swpafile later)

don't really need to separate `root` and `home`, unless you're very scared of things fucking up

make sure to change the type of the EFI partition from Linux filesystem to EFI (option `1`)

`mkfs.fat -F32 /dev/sdXY` for UEFI part, `mkfs.ext4 /dev/sdXZ` for other part(s)

Mount root partition using `mount /dev/sdXY /mnt` (probably `sda2`, or on laptop `nvme0n1p2`)

Create `/mnt/boot` (and optionally `/mnt/home`) directory (`mkdir /mnt/...`), and mount the EFI (and home) partition, respectively

Make sure the boot partition/mountpoint/dir is called `boot` not `efi`, since `pacstrap` will chuck your shit in `boot` regardless and it'll just cause you problems with `systemd-boot` down the line

`findmnt` to find mounted filesystems, check you've done it correctly

`umount -R` to recursively unmount a target and all its kids

### Install the system

`pacman -Sy reflector` in the USB system, and then run `reflector --latest 15 --protocol https --sort rate --save /etc/pacman.d/mirrorlist` to... do the thing

Install Arch and some basic text and networking utilities with `pacstrap /mnt base linux linux-firmware base-devel nano vi netctl networkmanager`

Run `genfstab -U /mnt >> /mnt/etc/fstab`, check the resulting file

### Enter the new system

`arch-chroot /mnt`

Run `ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime`, `timedatectl set-timezone Europe/London` and `hwclock --systohc`

Uncomment relevant locales in `/etc/locale.gen` and run `locale-gen`

`echo "LANG=en_GB.UTF8" >> /etc/locale.conf`

`echo "KEYMAP=uk" >> /etc/vconsole.conf`

`echo "[hostname]" >> /etc/hostname`

Edit the `/etc/hosts` file to say:

```
127.0.0.1   localhost
::1   localhost
127.0.1.1   [hostname].localdomain  [hostname]
```
Set root password with `passwd`

### Bootloader

#### Single-boot (eg laptop)

Install `efibootmgr`

Run `efibootmgr --disk /dev/[disk name] --part [boot partition number] --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=UUID=[boot partition UUID] silent initrd=\initramfs-linux.img' --verbose`

#### Dual-boot (eg PC)

`bootctl install`

`echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sdXY) rw" >> /boot/loader/entries/arch.conf` where `sdXY` is the root part (prob `sda2`)

Follow the instructions at [Loader Configuration](https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration) and [this site](https://www.addictivetips.com/ubuntu-linux-tips/set-up-systemd-boot-on-arch-linux/) to set up loader entries

`loader.conf` should have `auto-entries 0` and `auto-firmware 0`.

Make some sort of temp folder (eg `/windows_tmp`) and mount the windows boot partition (probably `/dev/sd_1`) to it. Copy over the Microsoft folder in the EFI to `/boot/EFI`, following the instructions [here](https://old.reddit.com/r/archlinux/comments/afki9z/dualboot_arch_and_windows_on_separate_drives_with/). Systemd-boot cannot detect efi boot managers from different drives, have to either mount both to the same partition or copy stuff over after the fact. 

You can then make a windows.conf with the lines `title Windows 10` and `efi /EFI/Microsoft/Boot/bootmgfw.efi`.

`bootctl update` to save everything

### Microcode

Install `amd-ucode` and add to `/boot/loader/entries/arch.conf` the line `initrd /amd-ucode.img` as the first initrd line (ie. before initramfs).

### Done

`exit` the chroot, `umount-R /mnt` just to be safe, `reboot` into the new system!

## Post-Install Config

- [General Recommendations](https://wiki.archlinux.org/index.php/General_recommendations)
- https://www.reddit.com/r/archlinux/comments/3ya67j/install_arch_infographic/
- https://wiki.archlinux.org/index.php/Laptop/Lenovo
- https://fhackts.wordpress.com/2018/12/28/arch-linux-on-a-lenovo-x280/

### Create non-root user 

```
useradd -m -G wheel [username] 
passwd [username]
visudo /etc/sudoers
```

Once in `/etc/sudoers`, uncomment the line allowing wheel users to run any command and save (press i, do stuff, press escape, `:wq`, press enter)

### Get rid of weird caching message

https://unix.stackexchange.com/questions/257270/get-rid-of-no-caching-mode-page-found-message-during-boot

### Networking

`systemctl enable NetworkManager`, `nmtui`

### Graphical environment

Ctrl+Alt+F2(3,4...) starts new tty

Alt+Left,Right to move ttys

Download the `xorg` package group, `mesa` and the relevant video drivers

Clone `dwm` + `st` from git repos, run `make` and `sudo make install`. install `dmenu`.

`echo "exec dwm" >> ~/.xinitrc`

Install a font, eg Hack

To find a font name: `fc-list | grep -i "keyword`

Then whack it into `~/st/config.h`

## //TODO

### ✅ get AUR helper `yay`

### 🔁 ensure arch boots properly and fast

using `systemd-analyze`, I found refind took at least 8 seconds to boot. Direct EFISTUB booting takes about 20ms. Not sure if I can make refind any faster, or if I can decrease the firmware, kernel or userspace t

### ❌ make a script to make changing EFISTUB boot using `efibootmgr` less painful

### ✅ `alsa`, `alsa-utils`, `alsa-packages` for sound management

### 🔁 check if it's my laptop speakers that sound shit or if I can configure sound to be better

(2020-05-25) Using my headphones I have ascertained that the speakers were shit and the audio output was fine. Still want to do some tweaking and EQ to make it as good as possible

- https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture
- https://wiki.archlinux.org/index.php/PulseAudio
- https://aur.archlinux.org/packages/ncpamixer/
- https://www.reddit.com/r/archlinux/comments/96lc2h/pulseaudio_poor_quality/
- https://medium.com/@gamunu/enable-high-quality-audio-on-linux-6f16f3fe7e1f
- https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture/Troubleshooting#Audio_Quality
- https://wiki.archlinux.org/index.php/PulseAudio/Troubleshooting

### ❌ configure output to DAC

### ✅ configure bluetooth

- make sure to `sudo bluetoothctl` if you want to pair and connect to speaker

### ❌ automate bluetooth connection

- https://wiki.archlinux.org/index.php/Bluetooth_headset#Configuration_via_CLI

### ✅ make sure `mesa` and relevant Intel drivers are present

- https://wiki.archlinux.org/index.php/Intel_graphics

### ✅ download `i3-wm` & extras, and initialise 

- add `exec i3` to `~/.xinitrc`

### ❌ change `xorg` keyboard config & check resolution/DPI

- https://wiki.archlinux.org/index.php/Xorg/Keyboard_configuration
- https://wiki.archlinux.org/index.php/Xorg#Display_size_and_DPI

### ❌ change terminal emulator from `rxvt-unicode` to something better

### ❌ customise terminal emulator

### ❌ decide between `bash` and `zsh`

### ❌ make cool shell aliases

### ❌ make `reflector` do its thing automatically at some interval

### ❌ touchpad drivers

### ❌ trackpoint control drivers

### 🔁 make sure Fn keys work

(2020-05-25) Audio Fn keys work out of the box - great stuff. I still need to make sure their volume is capped at 100 and edit the volume increment.

### ❌ disable Fn mute LEDs forever, if possible

### ❌ make swapfile

### ❌ make sure laptop goes to sleep properly

### ❌ ensure performance and battery conservation are balanced

- https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling
- https://wiki.archlinux.org/index.php/General_recommendations#Power_management

### ❌ check if network manager automatically switches over to ethernet when plugged in, and back to wifi when not

### 🔁 fuck around with fonts

Hack is ok for now, but I want something with a line through the 0, not a dot inside it. Also no ligatures, they're ugly.

### ❌ display manager for logging in, not having to `startx` through tty (or just put a script in `~/.profile`)

### ❌ locking my screen

### ❌ make a `~/.profile` file

### ❌ get a terminal browser

- https://www.brow.sh/

## Configure a terminal emulator

`st` appears to be the best of them, but it looks very difficult to customise. I can't seem to decide on a second-best option.

## Configure `dwm`

## Configure the Status Bar

### ❌ make it look nice

### ❌ volume controls

### ❌ music indicator + controls

### ❌ bluetooth controls

### ❌ battery indicator
