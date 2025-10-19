# Frankenender Klipper Config Git Setup

This system uses a **bare Git repository** to version only the printer configuration files on `frankenender`,
without touching other system files or the separate `klipper`, `moonraker`, or `fluidd` repositories.

---

## Modifications
- 235 x 230 x 245 Bed
- BlTouch V3.1 (28, -25) Offset
- SKR Mini E3 V2
- E3D V6 Hotend
- BMG Extruder (1400 mA)
- Custom Direct Drive (8, 14) Offset
- Probably other things i'm forgetting...

## Klipper
- RPI Zero W 2 Wired to USART1 (PA11 & PA12)
Fought with getting usart connected, [this](https://www.reddit.com/r/klippers/comments/v26f2e/cant_connect_to_skr_mini_e3_v2_over_uart/) helped
```
In RPI
/boot/cmdline.txt, delete this:
console=serial0,115200
Then in your shell:
echo "dtoverlay=disable-bt" | sudo tee -a /boot/config.txt
echo "enable_uart=1" | sudo tee -a /boot/config.txt
sudo systemctl stop serial-getty@ttyAMA0.service
sudo systemctl disable serial-getty@ttyAMA0.service
sudo reboot
2. Compile klipper UART
Then make and flash the new firmware.
- (RPI) TX <-> RX (Motherboard)
- (RPI) RX <-> TX (Motherboard)
- (RPI) Ground <-> Ground (Motherboard)
SKR MINI E3 V2.0
- STM32
- STM32F103
- BOOTLOADER 28KIB
- CLOCK REFERENCE 8 MHZ
- USART2 PA3/PA2
- BAUD 250000
- GPIO PINS STARTUP !PA14
printer.cfg, set:
[mcu]
serial: /dev/ttyAMA0
restart_method: command
```


## 📦 Repo Layout

Only the following directories are tracked:

- `~/printer_data/config/` — all Klipper configuration files and macros  
- `~/klipper-kconfigs/` — firmware Kconfig files for MCU builds  
- *(optional)* `~/fluidd-config/` — only if customized; otherwise ignored

Everything else (gcodes, logs, caches, databases, virtualenvs, etc.) is ignored.

---

## ⚙️ Setup Summary

### 1. Initialize the bare repo
```bash
git init --bare $HOME/.printercfg.git
git --git-dir=$HOME/.printercfg.git --work-tree=$HOME config status.showUntrackedFiles no
```

### 2. Create an alias for convenience
Add this to your shell rc file (`~/.bashrc`, `~/.zshrc`, or `~/.config/fish/config.fish`):

```bash
alias klipgit='git --git-dir=$HOME/.printercfg.git --work-tree=$HOME'
```

Then reload your shell:
```bash
source ~/.bashrc   # or the equivalent for your shell
```

Now you can use `klipgit` just like normal Git:
```bash
klipgit status
klipgit add printer_data/config klipper-kconfigs
klipgit commit -m "update configs"
klipgit push
```

### 3. Add and commit files
```bash
klipgit add .gitignore printer_data/config klipper-kconfigs
klipgit commit -m "Initial commit: Klipper configs + Kconfig files"
```

### 4. Add remote and push
```bash
klipgit remote add origin git@github.com:brycewalkerdev/frankenender-klipper.git
klipgit branch -M main
klipgit push -u origin main
```

---

## 🧹 `.gitignore` Example

```bash
/*                     # ignore everything by default
!.gitignore            # track this file
!printer_data/
printer_data/*
!printer_data/config/
!printer_data/config/**
!klipper-kconfigs/
!klipper-kconfigs/**
fluidd-config/          # ignore (unmodified upstream repo)
*.bak *.bkp *.old *.zip *.tar.gz *.bin
```

---

## 🪄 Optional Auto-Commit Helper

Create `~/bin/klip-sync`:

```bash
#!/usr/bin/env bash
set -e
git --git-dir="$HOME/.printercfg.git" --work-tree="$HOME" add printer_data/config klipper-kconfigs
git --git-dir="$HOME/.printercfg.git" --work-tree="$HOME" commit -m "${1:-update configs}"
git --git-dir="$HOME/.printercfg.git" --work-tree="$HOME" push
```

Make it executable:
```bash
chmod +x ~/bin/klip-sync
```

Now you can sync with:
```bash
klip-sync "Updated macros and PID tuning"
```

---

## 🧩 Notes

- `klipper/`, `moonraker/`, and `fluidd/` are **their own Git repos**. Don’t include them here.
- `printer_data/database/`, `gcodes/`, `logs/`, etc. contain large or ephemeral data and should not be versioned.

---

## 🪛 Restore / Fresh Install Instructions

To restore this configuration on a new Raspberry Pi or after reinstalling the OS:

### 1. Clone the bare repo
```bash
git clone --bare git@github.com:brycewalkerdev/frankenender-klipper.git $HOME/.printercfg.git
```

### 2. Create the alias
Add this to your rc file:
```bash
alias klipgit='git --git-dir=$HOME/.printercfg.git --work-tree=$HOME'
```
Then reload your shell:
```bash
source ~/.bashrc
```

### 3. Check out the tracked files
```bash
klipgit checkout
```

If any files already exist, back them up first. After checkout, your `printer_data/config/` and `klipper-kconfigs/` directories will be restored exactly as committed.

### 4. Verify everything
```bash
klipgit status
```

You should see a clean working tree.

### 5. Update and sync normally
Continue managing changes with:
```bash
klipgit add printer_data/config klipper-kconfigs
klipgit commit -m "Updated configs after restore"
klipgit push
```

---

**Maintainer:** Bryce Walker  
**System:** `frankenender`  
**Purpose:** Track persistent configuration and firmware build settings for Klipper, Moonraker, and Fluidd.

