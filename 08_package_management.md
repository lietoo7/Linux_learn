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
## 6. Installation Hors-Ligne (Offline)
Utile pour les environnements isolés (air-gap) via un support externe (clé USB, disque).
C'est une excellente idée. Sans cette étape, la section "Hors-ligne" est incomplète car l'utilisateur se retrouvera bloqué par une cascade de dépendances manquantes une fois devant sa machine isolée.
Voici la section complémentaire à insérer juste avant l'étape d'installation (ou au début de la section 6) pour rendre la fiche vraiment opérationnelle.
### 6.1. Préparation : Récupérer le paquet ET ses dépendances
Sur la machine connectée à Internet, avant le transfert sur clé USB.

Avec RHEL / AlmaLinux / CentOS (DNF)

La commande download avec l'option --resolve est la méthode la plus fiable pour ne rien oublier.
#### Créez un dossier propre pour le transfert
mkdir ~/transfert_usb && cd ~/transfert_usb

#### Télécharge le paquet et TOUTES ses dépendances non installées
sudo dnf download --resolve --alldeps <nom_du_paquet> --destdir=.

 * --resolve : Analyse l'arbre des dépendances.
 * --alldeps : Force le téléchargement de toutes les dépendances (même si elles sont déjà sur le PC connecté).

Avec Debian / Ubuntu (APT)

APT ne possède pas d'option native aussi directe que DNF pour le téléchargement groupé, mais on utilise souvent apt-get :
#### Télécharge les .deb dans le dossier courant
sudo apt-get download $(apt-rdepends <nom_du_paquet> | grep -v "^ " | grep -v "debconf")

Note : nécessite l'outil apt-rdepends (sudo apt install apt-rdepends).

Avec Arch Linux (Pacman)

Arch utilise son cache local ou des outils tiers, mais la méthode standard est :
sudo pacman -Sw <nom_du_paquet> --cachedir .

 * -Sw : Download only (ne pas installer).

> [!IMPORTANT]
> Compatibilité des versions : Pour que cette méthode fonctionne à 100 %, la machine connectée doit idéalement être sous le même OS (et la même version mineure) que la machine isolée. Si la machine connectée est sous Alma 9.4 et l'isolée sous Alma 9.1, il peut y avoir des conflits de versions de librairies critiques comme la glibc.
> 

### 6.2 Ensuite sur la Machine en offline
Debian / Ubuntu (.deb)
sudo dpkg -i <nom_du_paquet>.deb

> Note : Si des dépendances manquent (erreur de configuration), utilisez sudo apt --fix-broken install une fois les dépendances copiées localement.
> 
RHEL / AlmaLinux / CentOS (.rpm)
sudo dnf install --disablerepo=* --nogpgcheck *.rpm

 * --disablerepo=* : Force DNF à ignorer les dépôts distants (évite les erreurs de timeout).
 * --nogpgcheck : Ignore la signature GPG (nécessaire si les clés publiques n'ont pas été importées sur la machine isolée).
Arch Linux (.pkg.tar.zst)
sudo pacman -U /chemin/vers/paquet.pkg.tar.zst

### 6.3 La petite astuce avec Snap, Download & Install
La procédure en 2 étapes
 * Récupérer les fichiers (avec internet) :
   * Commande : snap download [nom]
   * Résultat : Vous obtenez deux fichiers : le logiciel (.snap) et sa signature de sécurité (.assert).
 * Installer les fichiers (sans internet) :
   * Étape A : sudo snap ack fichier.assert (Sert à valider l'authenticité du logiciel).
   * Étape B : sudo snap install fichier.snap (Lance l'installation physique).

Pourquoi faire cela ?
 * Mode Hors-ligne : Installer des logiciels sur un PC sans connexion.
 * Gain de temps : Télécharger une seule fois pour déployer sur plusieurs machines.
 * Sécurité : Vérifier l'intégrité du paquet avant de l'injecter sur un système critique.

> Point de vigilance : Pour qu'un Snap fonctionne sur un PC totalement vierge, assurez-vous d'avoir aussi téléchargé et installé le paquet de base (souvent core20 ou core22) via la même méthode.
> 

---
## 7. Vérification des Signatures et Intégrité
Procédure de sécurité pour s'assurer qu'un paquet n'a pas été corrompu ou altéré.
Vérification par somme de contrôle (Hash)
Comparez la valeur produite par cette commande avec celle fournie sur le site officiel :
sha256sum <nom_du_fichier>

Vérification par signature GPG
Vérifie que le paquet a été signé par une clé de confiance de l'éditeur.
 * RHEL / AlmaLinux :
   rpm --checksig <nom_du_paquet>.rpm

 * Debian / Ubuntu :
   dpkg-sig --verify <nom_du_paquet>.deb

 * Arch Linux : pacman gère cela nativement via la configuration SigLevel dans /etc/pacman.conf
---
## 8. Afficher Les paquets et dépendances 
### 8.1 Lister les paquets
| Distribution | Commande pour tout lister | Filtrer un paquet spécifique | Lister les paquets manuels |
|---|---|---|---|
| Debian / Ubuntu | dpkg -l | dpkg -l | grep <nom> | (Via apt) apt list --installed |
| RHEL / AlmaLinux | rpm -qa | rpm -qa | grep <nom> | dnf list installed |
| Arch Linux | pacman -Q | pacman -Q | grep <nom> | pacman -Qe |

Rappels utiles
 * Le Pipe (|) : Ton utilisation de grep est universelle. C'est la méthode la plus rapide pour ne pas polluer ton écran avec des milliers de lignes.
 * Précision Arch Linux : La commande pacman -Qe est particulièrement propre pour nettoyer un système, car elle ne montre que ce que tu as décidé d'installer, pas les centaines de bibliothèques (dépendances) qui viennent avec.

### 8.2 Afficher les dépendances d’un paquet
| Famille / Distro | Paquet déjà installé | Paquet avant installation | Note spécifique |
|---|---|---|---|
| Debian / Ubuntu | dpkg -s <nom> | apt-cache depends <nom> | dpkg est local uniquement. |
| RHEL / AlmaLinux | rpm -qR <nom> | dnf deplist <nom> | dnf interroge les dépôts. |
| Arch Linux | pacman -Qi <nom> | pacman -Si <nom> | -Q (Query) vs -S (Sync). |

Points clés à retenir
 * Arch Linux : La logique est très visuelle. Le suffixe -i (info) s'ajoute soit à -Q pour le local, soit à -S pour le distant.
 * Debian/Ubuntu : dpkg -s (status) donne beaucoup d'infos, dont les dépendances, mais seulement si le paquet est présent sur ton disque.
 * RHEL : dnf deplist est extrêmement détaillé, il liste même les capacités (capabilities) requises par le paquet.

---
## 9. Suppression de paquets sous Linux
1. Famille Debian / Ubuntu
| Outil | Commande | Particularité |
|---|---|---|
| APT | sudo apt remove <nom> | Supprime le logiciel uniquement. |
| APT (Complet) | sudo apt purge <nom> | Supprime aussi les fichiers de configuration. |
| dpkg | sudo dpkg -r <nom> | Pour les paquets .deb installés localement. |
2. Famille RHEL / AlmaLinux / Fedora
| Outil | Commande | Particularité |
|---|---|---|
| DNF | sudo dnf remove <nom> | Gère proprement les dépendances. |
| RPM | sudo rpm -e <nom> | Plus "brut", peut bloquer si des dépendances existent. |
3. Arch Linux
| Outil | Commande | Particularité |
|---|---|---|
| Pacman | sudo pacman -R <nom> | Suppression simple. |
| Pacman + Déps | sudo pacman -Rs <nom> | Supprime aussi les dépendances inutilisées. |
4. Formats Universels (Snap)
| Outil | Commande | Particularité |
|---|---|---|
| Snap | sudo snap remove <nom> | Par défaut, crée un snapshot (sauvegarde) des données. |
| Snap (Purge) | sudo snap remove <nom> --purge | Supprime tout, sans garder de snapshot. |
