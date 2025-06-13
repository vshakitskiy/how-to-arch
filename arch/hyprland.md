# WIP: Adding Hyprland to your Arch Linux installation

So, you've installed Arch Linux and you want to install Hyprland? This guide will help you do that.

## Prerequisites

- Arch Linux installed. (If not, please refer to the [my simple installation guide](./setup.md).)
- At least network capabilities.

## Installation

#### 1. Install Hyprland package with some basic packages

```bash
sudo pacman -S hyprland kitty ttf-dejavu
```

#### 2. Install Audio packages

```bash
sudo pacman -S pipewire pipewire-pulse pipewire-alsa wireplumber
```

#### 3. Install Bluetooth and Network Manager packages

```bash
sudo pacman -S bluez bluez-utils network-manager-applet
```

#### 4. Install application launcher and status bar

```bash
sudo pacman -S waybar wofi
```

#### 5. Install prefered fonts

In my case, I will use my beloved Monocraft Nerd Font. You can use any other font you want. Place .ttf file inside `~/.local/share/fonts` or even to system-wide fonts directory `/usr/local/share/fonts/ttc`. Don't forget to run `fc-cache -fv` to refresh the font cache.