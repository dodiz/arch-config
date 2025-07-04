# Arch setup

Table of contents

- [Network commands](#network-commands)
- [Grub, add windows option](#grub-add-windows-option)
- [Hyprland](#hyprland)
  - [Nvidia and start](#nvidia-and-start)
  - [Layout it](#layout-it)
  - [DM Ly (login)](#dm-ly-login)
  - [Wofi (program launcher)](#wofi-program-launcher)
  - [Waybar (Status bar)](#waybar-status-bar)
  - [Vscode](#vscode)
  - [Hyprpaper](#hyprland)
  - [pcmanfm](#pcmanfm-files-manager)

## Network commands

- `networkctl` - Show network status net devices

  - `networkctl status <device>` - Show status of a specific network device
  - `ip addr show <device>` - Show IP address of a specific device
  - ethernet interface file: `/etc/systemd/netowork/20-wired.network` used to get an IP from wired network

    ```
    [Match]
    Name=eno1

    [Network]
    DHCP=yes
    ```

- `iwctl` - interface to manage wireless devices
  - `iwctl known-networks list` - List known networks
- `dhcpcd` - get ip

## Grub, add windows option

In case you installed another EFI partition to boot arch, windows boot partition won't show anymore, so to add it to grub again

- run `sudo pacman -Syu os-prober`
- edit `/etc/default/grub`, add

  ```
  GRUB_DISABLE_OS_PROBER=false
  ```

- run `lsblk -f` and find Windows EFI partition
- make directory `mkdir -p /mnt/efi-win`
- mount it `sudo mount /dev/nvme0n1p1 /mnt/efi-win` (change `nvme0n1p1 with windows EFI partition name)
- verify that `/mnt/efi-win/EFI/Microsoft/Boot/bootmgr.efi` exists
- edit `sudo nano /etc/grub.d/40_custom` with

  ```
  menuentry "Windows" {
      insmod part_gpt
      insmod fat
      set root='hd0,gpt1'
      chainloader /EFI/Microsoft/Boot/bootmgfw.efi
  }
  ```

  `hd0, gpt` means first disk, first partition, `nvme0n1p1` ni this case, likely

- reload grub config `sudo grub-mkconfig -o /boot/grub/grub.cfg`
- OPTIONAL: select windows by default: open `sudo nano /etc/default/grub`
- set `GRUB_DEFAULT='Windows'`, same name as `menuentry` above
- reload grub config `sudo grub-mkconfig -o /boot/grub/grub.cfg`

## Hyprland

- Install kitty terminal emulator `sudo pacman -S kitty`
- Install hyprland `sudo pacman -S hyprland`

### Nvidia and start

- Install `nvidia drivers` `sudo pacman -S nvidia nvidia-utils nvidia-settings`
- run `sudo nano /etc/modprobe.d/nvidia.conf` and add
  ```
    options nvidia-drm modeset=1
  ```
- run `sudo mkinitcpio -P` to update initramfs with the new `nvidia-drm` setting
- update grub `sudo nano /etc/default/grub` add `nvidia-drm.modeset=1` to this line
  ```
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash ...HERE..."
  ```
- reload grub config: `sudo grub-mkconfig -o /boot/grub/grub.cfg`
- update hyprland conf `nano ~/.config/hypr/hyprland.conf`, by adding:
  ```
    env = WLR_NO_HARDWARE_CURSORS,1
    env = __GLX_VENDOR_LIBRARY_NAME,nvidia
    env = WLR_EGL_NO_MODIFIERS,1
  ```
- test hyprland by running `dbus-run-session Hyprland`

### Layout it

- `sudo nano ~/.config/hypr/hyprland.conf`, edit
  ```
  input {
    kb_layout = it
  }
  ```

### DM Ly (login)

- `sudo pacman -S ly`
- `sudo systemctl enable ly.service`

- Make so Ly starts Hyprland using nvidia, `nano ~/.xsession`
- add this line `exec dbus-run-session Hyprland`
- make it executable `chmod +x ~/.xsession`
- check on kitty if `echo $DBUS_SESSION_BUS_ADDRESS` returns `unix:path=/run/user/1000/bus`

### Wofi (program launcher)

- `sudo pacman -S wofi`
- add `.config/wofi/style.css` from `https://github.com/dracula/wofi` or other

### Waybar (Status bar)

- `sudo pacman -S waybar`
- `sudo pacman -S ttf-font-awesome` for icons on waybar
- Icons: `sudo pacman -S ttf-nerd-fonts-symbols`

#### Waybar things

- `sudo pacman -S wlogout` for logout button

### Vscode

- run `yay -S visual-studio-code-insiders-bin`
- add `vscode/code-flags.conf` to `.config/code-flags.conf` for wayland support

### Hyprpaper (wallpaper)

- run `sudo pacman -S hyprpaper`
- add wallpapers in `.wallpaper` folder and configure `config/hyprpaper.conf`

### pcmanfm (Files manager)

- install pcmanfm-gtk3 `sudo pacman -S pcmanfm-gtk3`
- install lxapperance `sudo pacman -S lxappearance` to change theme
- install theming `yay -S arc-gtk-theme`
- copy `hyprland.conf` filer

### Mako (notification daemon)

- `sudo pacman -S mako`
- `sudo systemctl enable mako.service`

### Screen capture

- `sudo pacman -S grim slurp wl-clipboard swappy jq`

## TODOS

### git auto configure (password storage)

#### Waybar bluetooth manager

#### Waybar network manager
