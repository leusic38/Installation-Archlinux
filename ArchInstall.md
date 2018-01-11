# Install Arch linux uefi(sur ssd et dd):

   ## Préparation des disques

1. Changer clavier: en Français : touches: loqdkeys fr  ==>loadkeys fr
2. Partitionner les disques:

    Pour ma part j'ai un SSD pour le system (sda) et un DD pour home et le swap.
    Utilisation de gdisk
   1. ssd (/dev/sda) 
      1. boot (/dev/sda1) +1024M ef00
      2. / linuxsystem (/dev/sda2) tout par default
   2. dd (/dev/sdb)
      1. swap (/dev/sdb1)+32G 2x la taille de la mémoire vive 8200
      2. /home (/dev/sdb2) tout par default.

    on applique ce qu'on vient de créer:
    ```
    mkfs.ext4 /dev/sda2
    mkfs.fat -F32 /dev/sda1
    mkfs.ext4 /dev/sdb2
    ```
    * le swap:
       ```
       mkswap /dev/sdb1
       swapon /dev/sdb1
       ```

3. on monte les partitions:
   ```
    mount /dev/sda2 /mnt
    mkdir /mnt/{boot,home}
    mount /dev/sda1 /mnt/boot
    mount /dev/sdb2 /mnt/home
   ```
4. en cas de serveur proxi on ajoute ces commandes:
   ```
   export http_proxy=http://leproxy:leport/
   export https_proxy=$http_proxy
   export ftp_proxy=$http_proxy
   ```

## Passons à l'installation d'Archlinux:
  
   1. Installer les packages de base d'arch
      ```
      pacstrap /mnt base base-devel
      pacstrap /mnt zip unzip p7zip vim mc alsa-utils mtools dosfstools git lsb-release ntfs-3g exfat-utils
      ```
   2. création du fichier de listing des partitions:
      ```
      genfstab -U -p /mnt >> /mnt/etc/fstab
      ```
   3. installation de grub2:
      ```
      pacstrap /mnt grub os-prober efibootmgr
      ```

   4. Reglage de archlinux:
      1. On modifie la racine de  notre arch:
         ```
         arch-chroot /mnt
         ```
      2. creer le fichier: _/etc/vconsole.conf_ pour avoir le bon clavier, pour moi le français, et y insérer:
         ```
         KEYMAP=fr-latin9
         FONT=lat9w-16
         ```
      3. Création du fichier _/etc/locale.conf_ pour la langue francçaise ():
         ```
         LANG=fr_FR.UTF-8
         LC_COLLATE=C
         ```
      4. On vérifie que les lignes fr_FR.UTF-8 UTF-8 et en_US.UTF-8 UTF-8 du fichier _/etc/locale.gen_ sont décommentés et on génère des traductions:
         ```
         locale-gen
         export LANG=fr_FR.UTF-8
         ```
      5. On rentre le nom de son ordi dans le fichier _/etc/hostname_.
      6. Réglage du fuseau horaire:
         ```
         ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
         ```

      7. Si on est monoboot ou multiboot sans windows on peut appliquer l'heure utc:
      ```
      hwclock --systohc --utc
      ```
    5. Generation du grub(si lts remplacer linux par linux-lts):
       ```
       mkinitcpio -p linux
       grub-mkconfig -o /boot/grub/grub.cfg
       ```
       verification de l'uefi : si dans le resultat de la comande _mount_ on a:
          ```
          efivars on /sys/firmware/efi/efivars type efivars (rw,nosuid,nodev,noexec,relatime)
          ```
          on execute :
          ```
          grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=arch_grub --recheck
          ```
          ou arch_grub est un nom personnel reconaisssable qui sera utilisé pour créer le repertoire où le chargeur sera placé sous _/boot/EFI/_
          si non on la précède de :
          ```
          mount -t efivarfs efivarfs /sys/firmware/efi/efivarfs
          ```

          On creer une sauvegarde de notre grub par securité :
          ```
          mkdir /boot/EFI/boot
          cp /boot/EFI/arch_grub/grubx64.efi /boot/EFI/boot/bootx64.efi
          ```
    6. Creation du password root:
       ```
       passwd root
       ```
    7. installation de networkManager
       ```
       pacman -Syy networkmanager
       systemctl enable NetworkManager
       ```
    8. Installation des différents paquets important:
       * ntp pour la synchronisation de l'horloge.
       * cronie pour les taches répétitives.
       * les paquets gst pour le multimedia.
       * xorg pour l'affichage (penser au paquets des chipset graphique: )
       * les pilotes pour l'impressions: vérifiersi il existe despaquets dédié a votre imprimante(ex: hplip pour les hp).
       * les fonts
         ```
         pacman -S ntp cronie
         pacman -S gst-plugins-{base,good,bad,ugly} gst-libav
         pacman -S xorg-{server,xinit,apps} xf86-input-{mouse,keyboard} xdg-user-dirs
         pacman -S cups gimp gimp-help-fr python-pyqt5 foomatic-{db,db-ppds,db-gutenprint-ppds,db-nonfree,db-nonfree-ppds} gutenprint
         pacman -S  ttf-{bitstream-vera,liberation,freefont,dejavu}
         ```
    9. pour ma part j'utilise zsh comme shell avec oh-my-zsh et powerline theme au lieu de bash donc autant modifier de suite le shell. je le configurerai plus tard :
       ```
       pacman -S zsh
       chsh -s /bin/zsh
       ```
    10. Ajout d'un utilisateur:
        ```
        useradd -m -g wheel -c 'Nom complet de l’utilisateur' -s /bin/bash 'login utilisateur'
        passwd 'login utilisateur'
        ```
    11. autoriser sudo pour les utilisateurs(on ne travaille jamais en root):
        ```
        visudo
        ```

        décommenté la ligne "#%wheel ALL=(ALL) ALL" qui suit la ligne:
        ```
        ## Uncomment to allow members of group wheel to execute any command
        %wheel ALL=(ALL) ALL
        ```
    12. 