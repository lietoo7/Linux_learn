# Fiche 4 : Common Directories

Contrairement à Windows où chaque disque a sa propre lettre (`C:`, `D:`), Linux utilise une arborescence unique qui commence à la racine, représentée par le symbole **`/`** (root).

## 1. La Racine et les Binaires

* **`/`** : Le répertoire racine. Tout le système se trouve ici.
* **`/bin`** : Contient les commandes essentielles utilisables par tous les utilisateurs (ex: `ls`, `cp`, `cat`).
* **`/sbin`** : Binaires système essentiels, généralement réservés à l'administrateur (ex: `reboot`, `fdisk`).
* **`/usr/bin`** : La majorité des programmes utilisateur (ex: `python`, `git`, `curl`).

## 2. Configuration et Données Utilisateur

* **`/etc`** : **Le centre névralgique.** Contient tous les fichiers de configuration du système et des services (ex: `/etc/passwd` pour les utilisateurs, `/etc/ssh` pour SSH).
* **`/home`** : Contient les dossiers personnels des utilisateurs (ex: `/home/alice`). C'est ici que vous stockez vos documents et réglages personnels.
* **`/root`** : Le dossier personnel de l'utilisateur "root" (l'administrateur). Ce n'est **pas** la même chose que la racine `/`.

## 3. Fichiers Temporaires et Variables

* **`/tmp`** : Fichiers temporaires. Attention : le contenu de ce dossier est souvent supprimé à chaque redémarrage.
* **`/var`** : Données variables dont la taille change souvent.
* `/var/log` : Contient tous les fichiers de journaux (logs).
* `/var/www` : Souvent utilisé pour stocker les fichiers d'un site web.
* `/var/mail` : Boîtes de réception des emails locaux.



## 4. Matériel et Système (Pseudo-fichiers)

Sous Linux, "tout est fichier". Ces dossiers ne contiennent pas de fichiers réels sur le disque, mais des interfaces avec le noyau et le matériel.

* **`/dev`** : Fichiers de périphériques (ex: `/dev/sda` pour le disque dur, `/dev/random`).
* **`/proc`** : Informations sur les processus en cours et les ressources système (RAM, CPU).
* **`/sys`** : Informations sur le matériel et les pilotes (drivers).

## 5. Bibliothèques et Montage

* **`/lib`** & **`/lib64`** : Bibliothèques partagées indispensables au démarrage et aux binaires de `/bin` et `/sbin`.
* **`/mnt`** : Point de montage temporaire pour les systèmes de fichiers (ex: une partition de données).
* **`/media`** : Point de montage automatique pour les supports amovibles (clés USB, CD-ROM).
* **`/opt`** : Utilisé pour l'installation de logiciels propriétaires ou tiers qui ne suivent pas la hiérarchie standard.

---

## Résumé pratique

> **Où chercher ?**
> * Je veux modifier la config réseau : `/etc`
> * Je cherche pourquoi mon serveur a planté : `/var/log`
> * Je veux stocker un script personnel : `/home/votre_nom`
> * Je veux voir si ma clé USB est reconnue : `/dev` ou `/media`
 
