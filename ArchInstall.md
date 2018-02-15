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

   0. Modifier la liste des mirroirs(on cherche les fr :
      ```
      nano /etc/pacman.d/mirrorlist
      ```
  
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
    8. Démontage des partitions et redémarage:
       ```
       exit
       unmount -R /mnt
       reboot
       ```
    9. Installation des différents paquets important:
       * ntp pour la synchronisation de l'horloge.
       * cronie pour les taches répétitives.
       * les paquets gst pour le multimedia.
       * xorg pour l'affichage (penser au paquets des chipset graphique: )
          ```
          pacman -S xf86-video-vesa
          ```
       * (si test sur virtualBox ajouter "virtualbox-guest-utils" :
          ```
          pacman -S virtualbox-guest-utils
          ```
          choisir la seconde option quia deja les paquets compilés. puis on l'active:
          ```
          systemctl enable vboxservice
          ```
         )
       * les pilotes pour l'impressions: vérifier si il existe des paquets dédié a votre imprimante(ex: hplip pour les hp).
       * les fonts
         ```
         pacman -S ntp cronie
         pacman -S gst-plugins-{base,good,bad,ugly} gst-libav
         pacman -S xorg-{server,xinit,apps,xrand,} xf86-input-{mouse,keyboard} xdg-user-dirs
         pacman -S cups python-pyqt5 foomatic-{db,db-ppds,db-gutenprint-ppds,db-nonfree,db-nonfree-ppds} gutenprint
         pacman -S  ttf-{bitstream-vera,liberation,freefont,dejavu}
         ```
    10. pour ma part j'utilise zsh comme shell avec oh-my-zsh et powerline theme au lieu de bash donc autant modifier de suite le shell. je le configurerai plus tard :
        ```
        pacman -S zsh
        chsh -s /bin/zsh
        ```
    11. Ajout d'un utilisateur:
        ```
        useradd -m -g wheel -c 'Nom complet de l’utilisateur' -s /bin/zsh 'login utilisateur'
        passwd 'login utilisateur'
        ```
    12. autoriser sudo pour les utilisateurs(on ne travaille jamais en root):
        ```
        visudo
        ```

        décommenté la ligne "#%wheel ALL=(ALL) ALL" qui suit la ligne:
        ```
        ## Uncomment to allow members of group wheel to execute any command
        %wheel ALL=(ALL) ALL
        ```
    13. installatin des differents packages:
       ```
       pacman -S ranger firefox firefox-i18n-fr libreoffice-still-fr vlc urxvt
## Mise en place des interfaces graphiques:
En ce qui me concerne I3 et xfce4.

On peut maintenant tout faire sans etre root.

 1. Pour commencer installation d'une interface de connexion graphique:
    * installation de lightdm et lightdm-gtk-greeter
    ```
    sudo pacman -S lightdm lightdm-gtk-greeter
    ```
    * pour etre sur de bien avoir le clavier dans la langue fr:
    ```
    sudo localectl set-x11-keymap fr
    ```
 2. Installation de i3