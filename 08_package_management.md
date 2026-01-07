# Fiche 8 : Package Management

Sur Linux, on n'installe pas (généralement) des logiciels en téléchargeant un `.exe` sur internet. On utilise un **gestionnaire de paquets** qui pioche dans des dépôts officiels sécurisés.

## 1. Concepts Clés

* **Paquet (Package)** : Une archive contenant le logiciel, ses fichiers de configuration et la liste de ses dépendances.
* **Dépendances** : Autres bibliothèques ou logiciels nécessaires au fonctionnement du paquet principal.
* **Dépôt (Repository)** : Un serveur distant (comme un "App Store") où sont stockés les paquets.

---

## 2. Debian / Ubuntu (Gestionnaire : APT)

C'est la famille la plus courante (Debian, Ubuntu, Linux Mint, Kali).

* **Mettre à jour la liste des dépôts** (à faire avant d'installer) :
`sudo apt update`
* **Mettre à jour tous les logiciels installés** :
`sudo apt upgrade`
* **Installer un paquet** :
`sudo apt install <nom_du_paquet>`
* **Supprimer un paquet** (conserve la config) :
`sudo apt remove <nom_du_paquet>`
* **Supprimer un paquet ET sa configuration** :
`sudo apt purge <nom_du_paquet>`
* **Chercher un paquet** :
`apt search <mot_cle>`

---

## 3. RHEL / CentOS / Fedora (Gestionnaire : DNF / YUM)

Utilisé dans le monde professionnel et serveur.

* **Installer un paquet** :
`sudo dnf install <nom_du_paquet>`
* **Mettre à jour le système** :
`sudo dnf upgrade`
* **Supprimer un paquet** :
`sudo dnf remove <nom_du_paquet>`

---

## 4. Arch Linux (Gestionnaire : Pacman)

Connu pour sa simplicité technique et ses mises à jour continues (*Rolling Release*).

* **Synchroniser et mettre à jour tout le système** :
`sudo pacman -Syu`
* **Installer un paquet** :
`sudo pacman -S <nom_du_paquet>`
* **Supprimer un paquet et ses dépendances inutiles** :
`sudo pacman -Rs <nom_du_paquet>`

---

## 5. Formats Universels (Snap & Flatpak)

Pour pallier les différences entre distributions, des formats "universels" existent. Ils embarquent toutes leurs dépendances dans un environnement isolé (sandbox).

* **Snap** (créé par Canonical) : `sudo snap install vlc`
* **Flatpak** (très populaire sur desktop) : `flatpak install flathub org.videolan.VLC`

---

## Résumé pratique

> **La règle d'or :**
> N'installez jamais un logiciel avec `sudo` si vous ne savez pas d'où il vient. Privilégiez toujours les dépôts officiels de votre distribution via `apt`, `dnf` ou `pacman`.

 
