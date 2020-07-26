# kmlhpk's ***Musings on Arch***
## General Thoughts

I first installed Arch on a Lenovo ThinkPad X280 on the 2020-05-23.

Following the Arch Wiki and asking Alex for help where I couldn't figure something out myself was much better for a complete beginner like myself than my brief attempt at trying to install Manjaro via Architect. The Manjaro wiki is simply not good enough. Sure, Arch installation and basic config took the best part of like, 10 hours. But it was damn fun, and educational too. Though, knowing what I know now, using Architect would probably be a doddle.

After an extended hiatus (mostly due to laziness), I decided to reinstall Arch from scratch on said laptop on 2020-07-26. This was to re-familiarise myself with the process, to document it all properly as I went along, and to prepare myself for installing Arch on the new SSD I bought for my PC.

## Install Process

Following the [Installation Guide](https://wiki.archlinux.org/index.php/installation_guide)

`ip link`, `iwctl`, `wifi-menu` to get connected to the internet

### Partitions, Filesystems, Mounting

`parted` to start the partitioning process

#### Partitioning a disk

Coming Soon™

#### Creating filesystems

`lsblk -f` to list filesystems

`findmnt` to find mounted filesystems

`umount -R` to recursively unmount a target and all its kids

`mkfs.fat -F32 [target]` for UEFI part, `mkfs.ext4 [target]` for other parts

#### Mounting filesystems

Mount root (`/`) partition (eg `nvme0n1p2` on laptop) to `/mnt` using `mount /dev/nvme0n1p2 /mnt`

Create `/mnt/efi` and `/mnt/home` directories (`mkdir`), and mount the EFI and home partitions to them, respectively

### Installing the system

`pacman Sy reflector` in the USB system, and then run `reflector --latest 15 --protocol https --sort rate --save /etc/pacman.d/mirrorlist` to... do the thing

Install Arch and some basic text and networking utilities with `pacstrap /mnt base linux linux-firmware nano netctl networkmanager` 

Run `genfstab -U /mnt >> /mnt/etc/fstab`, check the resulting file

Run `ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime`, `timedatectl set-timezone Europe/London` and `hwclock --systohc`

`KEYMAP=uk` into `/etc/vconsole.conf` (can also `loadkeys uk` if needed on live medium)

Install refind, and then edit `/boot/refind_linux.conf` to say something like `"boot with standard options" "root=UUID=XXX...."`


## Post-Install Config

- [General Recommendations](https://wiki.archlinux.org/index.php/General_recommendations)
- https://www.reddit.com/r/archlinux/comments/3ya67j/install_arch_infographic/
- https://wiki.archlinux.org/index.php/Laptop/Lenovo
- https://fhackts.wordpress.com/2018/12/28/arch-linux-on-a-lenovo-x280/

### ❌ edit refind config file with UUID kernel params

- https://wiki.archlinux.org/index.php/REFInd#Installation_with_refind-install_script
- https://wiki.archlinux.org/index.php/Kernel_parameters#rEFInd
- https://wiki.archlinux.org/index.php/Microcode
- https://old.reddit.com/r/archlinux/comments/a5ppnb/what_should_my_refind_linuxconf_look_like_how_do/

### ✅ get AUR helper `yay`

### ❌ ensure arch boots properly and fast

### ❌ make refind boot menu pretty

### ✅ create non-root user 

- `useradd -m -G wheel [username]` 
- `passwd [username]`
- `visudo /etc/sudoers` to reflect changes

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

### ✅ download relevant `xorg` packages

### ✅ make sure `mesa` and relevant Intel drivers are present

- https://wiki.archlinux.org/index.php/Intel_graphics

### ✅ download `i3-wm` & extras, and initialise 

- add `exec i3` to `~/.xinitrc`

### ❌ change `xorg` keyboard config & check resolution/DPI

- https://wiki.archlinux.org/index.php/Xorg/Keyboard_configuration
- https://wiki.archlinux.org/index.php/Xorg#Display_size_and_DPI

### ❌ change terminal emulator from `rxvt-unicode` to something better

- https://www.google.com/search?hl=en&q=linux%20best%20terminal%20emulators

### ❌ customise terminal emulator

- https://addy-dclxvi.github.io/post/configuring-urxvt/
- https://wiki.archlinux.org/index.php/Rxvt-unicode#Configuration

### ❌ decide between `bash` and `zsh`

### ❌ make cool shell aliases

### ❌ make `reflector` do its thing automatically at some interval

### ❌ touchpad drivers

### ❌ trackpoint control drivers

### 🔁 make Fn keys work

(2020-05-25) Audio Fn keys work out of the box - great stuff. I still need to make sure their volume is capped at 100 and edit the volume increment.

### ❌ disable Fn mute LEDs forever, if possible

### ❌ make swapfile

### ❌ make sure laptop goes to sleep properly

### ❌ ensure performance and battery conservation are balanced

- https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling
- https://wiki.archlinux.org/index.php/General_recommendations#Power_management

### ❌ check if network manager automatically switches over to ethernet when plugged in, and back to wifi when not

### ❌ fuck around with fonts

### ❌ display manager for logging in, not having to `startx` through tty (or just put a script in `~/.profile`)

### ❌ locking my screen

### ❌ make a `~/.profile` file

### ❌ get a terminal browser

- https://www.brow.sh/

## Making i3 Look Nice

## Configuring the Status Bar

### ❌ make it look nice

### ❌ volume controls

### ❌ music indicator + controls

### ❌ bluetooth controls

### ❌ battery indicator