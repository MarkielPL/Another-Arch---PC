# AnotherArch-PC

To jest skonsolidowana instrukcja instalacji Arch Linux w jednym języku: po polsku.

## 1. Weryfikacja UEFI
Upewnij się, że system live bootował w trybie UEFI:

<<<<<<< HEAD
*iwctl*
```
=======
```sh
ls /sys/firmware/efi/efivars
```

Jeśli katalog istnieje, jesteś w trybie UEFI.

## 2. Sprawdzenie sieci
Przetestuj połączenie z internetem:

```sh
ping -c 3 archlinux.org
```

Włącz synchronizację czasu:

```sh
timedatectl set-ntp true
```

## 3. Sieć Wi-Fi
Odblokuj interfejsy radiowe i połącz się z siecią:

```sh
rfkill unblock all
```

Uruchom `iwctl` i wykonaj polecenia:

```sh
iwctl
>>>>>>> 82d888c (Update README translations)
device list
station <INTERFACE> get-networks
station <INTERFACE> connect <SSID>
```

Jeśli sieć jest ukryta:

```sh
station <INTERFACE> connect-hidden <SSID>
exit
```

## 4. Inicjalizacja kluczy pacmana
Po połączeniu z internetem zainicjalizuj klucze pacmana:

```sh
pacman-key --init
pacman-key --populate archlinux
```

- `pacman-key --init` tworzy lokalną bazę kluczy GPG.
- `pacman-key --populate archlinux` importuje oficjalne klucze podpisywania pakietów.

## 5. Opcje instalacji

### Opcja A: `archinstall`

```sh
archinstall
```

- Automatyczny instalator Arch Linux.
- Przydatny, gdy chcesz szybko zainstalować system bez ręcznego konfigurowania.

### Opcja B: `archfi`

```sh
curl -L archfi.sf.net/archfi > archfi
sh archfi
```
<<<<<<< HEAD
https://github.com/MatMoul/archfi
## option *C* - manual instalation:
lsblk
>cfdisk /dev/...
=======
>>>>>>> 82d888c (Update README translations)

- Skrypt instalacyjny dla Arch Linux.
- Repozytorium: https://github.com/MatMoul/archfi

### Opcja C: instalacja ręczna

1. Sprawdź dostępne dyski i partycje:
   ```sh
   lsblk
   ```
2. Uruchom `cfdisk` na docelowym dysku:
   ```sh
   cfdisk /dev/sdX
   ```
3. Przykładowy układ partycji:
   - `/dev/sdX1` — EFI 512M, FAT32
   - `/dev/sdX2` — root 60G ext4 lub btrfs
   - `/dev/sdX3` — swap 16G
   - `/dev/sdX4` — /home pozostała przestrzeń

   Zastąp `sdX` odpowiednią literą dysku.

4. Utwórz systemy plików:
   ```sh
   mkfs.fat -F32 /dev/sdX1
   mkfs.btrfs /dev/sdX2
   mkfs.btrfs /dev/sdX4
   mkswap /dev/sdX3
   swapon /dev/sdX3
   ```

   - Jeśli wolisz ext4 dla `/`, użyj:
     ```sh
     mkfs.ext4 /dev/sdX2
     ```

5. Zamontuj partycje:
   ```sh
   mount /dev/sdX2 /mnt
   mkdir -p /mnt/{boot,home}
   mount /dev/sdX1 /mnt/boot
   mount /dev/sdX4 /mnt/home
   ```

6. Zainstaluj podstawowy system:
   ```sh
   pacstrap /mnt base base-devel linux linux-firmware nano usbutils amd-ucode btrfs-progs
   ```

   - Dla procesora Intel użyj `intel-ucode` zamiast `amd-ucode`.

7. Wygeneruj `fstab` i przejdź do chroot:
   ```sh
   genfstab -U /mnt >> /mnt/etc/fstab
   arch-chroot /mnt
   ```

## 6. Konfiguracja systemu w `arch-chroot`

1. Ustaw strefę czasową:
   ```sh
   ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
   hwclock --systohc --utc
   ```

2. Włącz lokalizacje:
   ```sh
   echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
   echo "pl_PL.UTF-8 UTF-8" >> /etc/locale.gen
   locale-gen
   ```

3. Ustaw język systemowy:
   ```sh
   echo "LANG=en_US.UTF-8" > /etc/locale.conf
   ```

4. Skonfiguruj konsolę:
   ```sh
   cat > /etc/vconsole.conf <<'EOF'
KEYMAP=pl
FONT=Lat2-Terminus16.psfu.gz
EOF
   ```

5. Ustaw nazwę hosta:
   ```sh
   echo "mojhost" > /etc/hostname
   ```

6. Skonfiguruj `/etc/hosts`:
   ```sh
   cat > /etc/hosts <<'EOF'
127.0.0.1 localhost.localdomain localhost
::1       localhost.localdomain localhost
127.0.1.1 mojhost.localdomain mojhost
EOF
   ```

   Zamień `mojhost` na swoją nazwę hosta.

7. Utwórz initramfs i ustaw hasło root:
   ```sh
   mkinitcpio -P
   passwd
   ```

8. Dodaj użytkownika:
   ```sh
   useradd -m -g users -G wheel,storage,power -s /bin/bash -d /home/<uzytkownik> <uzytkownik>
   passwd <uzytkownik>
   ```

   Zastąp `<uzytkownik>` realną nazwą użytkownika.

## 7. Instalacja sieci i bootloadera

1. Zainstaluj NetworkManager:
   ```sh
   pacman -S networkmanager
   systemctl enable NetworkManager
   ```

   Po restarcie możesz połączyć się z Wi-Fi:
   ```sh
   nmcli device wifi connect <SSID> password <PASSWORD>
   ```

2. Wybierz bootloader:

### systemd-boot
   ```sh
   pacman -S --needed efibootmgr dosfstools
   bootctl --path=/boot install
   ```

   Utwórz `/boot/loader/loader.conf`:
   ```ini
default arch
timeout 3
console-mode max
editor no
   ```

   Utwórz `/boot/loader/entries/arch.conf`:
   ```ini
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/sdX2 rw
   ```

### GRUB
   ```sh
   pacman -S --needed grub efibootmgr os-prober
   grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

### rEFInd
   ```sh
   pacman -S --needed refind
   refind-install
   ```

   Opcjonalnie edytuj `/boot/EFI/refind/refind.conf` dla niestandardowych ścieżek do jądra lub initramfs.

3. Zakończ instalację:
   ```sh
   exit
   umount -R /mnt
   reboot
   ```

## 8. Po restarcie

1. Zaktualizuj system:
   ```sh
   pacman -Syu
   ```

2. Włącz `multilib` w `/etc/pacman.conf`:
   ```ini
[multilib]
Include = /etc/pacman.d/mirrorlist
   ```

3. Zsynchronizuj bazy pakietów:
   ```sh
   pacman -Syu
   ```

4. Skonfiguruj mirrorlist:
   ```sh
   pacman -S reflector rsync curl
   reflector --verbose --country "your country" --age 12 --sort rate --save /etc/pacman.d/mirrorlist
   ```

   Lub:
   ```sh
   reflector --verbose --latest 200 --age 12 --protocol http,https --sort rate --save /etc/pacman.d/mirrorlist
   ```

5. Zainstaluj `yay`:
   ```sh
   pacman -S git
   cd /opt
git clone https://aur.archlinux.org/yay-git.git
chown -R $USER:$USER yay-git
cd yay-git
makepkg -si
   ```

6. Zainstaluj dodatkowe firmware z AUR:
   ```sh
   git clone https://aur.archlinux.org/wd719x-firmware.git
   cd wd719x-firmware
   makepkg -sri
   ```

   lub:
   ```sh
   git clone https://aur.archlinux.org/aic94xx-firmware.git
   cd aic94xx-firmware
   makepkg -sri
   ```

7. Zainstaluj sterowniki:
   ```sh
   yay -S upd72020x-fw
   ```

   Jeśli trzeba, przebuduj initramfs:
   ```sh
   mkinitcpio -p linux
   ```

8. Zainstaluj przykładowe pakiety:
   ```sh
   yay -S firefox yakuake
   ```

## 9. Środowisko graficzne i konfiguracja

### Hyprland / konfiguracje
- https://github.com/JaKooLit
- https://github.com/JaKooLit/Arch-Hyprland
- https://github.com/end-4/dots-hyprland/tree/main\?tab\=readme-ov-file

### KDE
- https://tuxinit.com/minimal-kde-plasma-install-arch-linux/
- https://wiki.archlinux.org/title/KDE

Zainstaluj Xorg i KDE:
   ```sh
   yay -S xorg xorg-xinit firefox plasma-nm plasma-pa dolphin konsole kdeplasma-addons yakuake
   ```

Włącz SDDM:
   ```sh
   systemctl enable sddm
   ```

## 10. Stylizacja terminala

1. Zainstaluj czcionki:
   ```sh
   yay -S ttf-nerd-fonts-symbols
   ```
2. Zainstaluj Oh My Zsh i Powerlevel10k:
   ```sh
   sh -c "$(wget https://raw.github.com/ohmyzsh/oh-my-zsh/master/tools/install.sh -O -)"
   git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
   ```
3. Zainstaluj pluginy:
   ```sh
   git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
   git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
   ```
4. Edytuj `~/.zshrc`:
   - ustaw `ZSH_THEME="powerlevel10k/powerlevel10k"`
   - ustaw `plugins=(git zsh-autosuggestions zsh-syntax-highlighting)`
   - odkomentuj `ENABLE_CORRECTION="true"`

## Uwagi
- Zastąp `/dev/sdX`, `/dev/sdX1`, `/dev/sdX2`, `/dev/sdX3`, `/dev/sdX4` właściwymi urządzeniami.
- Użyj `amd-ucode` lub `intel-ucode` zgodnie z procesorem.
- `base-devel` jest potrzebny do budowania pakietów z AUR.
