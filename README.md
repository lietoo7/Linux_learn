# Linux Fundamentals
---

## Sommaire

1. [Interaction avec le système de fichiers]( )
2. [Shell Operators]( )
3. [Permissions 101]( )
4. [Common Directories]( )
5. [Processes 101]( )
6. [Automation]( )
7. [System Logs]( )
8. [Package Management]( )

---

## 1. Interaction avec le système de fichiers

| Commande | Description |
| --- | --- |
| `pwd` | Affiche le chemin du répertoire actuel (*Print Working Directory*). |
| `ls` | Liste le contenu d'un répertoire (`-la` pour voir les fichiers cachés et détails). |
| `cd` | Change de répertoire courant. |
| `cat` | Affiche le contenu complet d'un fichier dans le terminal. |
| `touch` | Crée un fichier vide ou met à jour la date de modification. |
| `mkdir` | Crée un nouveau dossier. |
| `cp` | Copie des fichiers ou dossiers (`-r` pour le récursif). |
| `mv` | Déplace ou renomme un fichier/dossier. |
| `rm` | Supprime un fichier (`-rf` pour forcer la suppression d'un dossier). |
| `find` | Recherche des fichiers selon des critères (nom, taille, date). |
| `grep` | Cherche un motif textuel spécifique dans un fichier ou un flux. |
| `wc` | Compte le nombre de lignes (`-l`), mots ou caractères d'un fichier. |
| `file` | Détermine le type de contenu d'un fichier (ex: texte, exécutable, image). |

---

## 2. Shell Operators

Les opérateurs permettent de manipuler les entrées/sorties et de chaîner les commandes.

* `>` : Redirige la sortie vers un fichier (écrase le contenu).
* `>>` : Redirige la sortie vers un fichier (ajoute à la fin).
* `|` (Pipe) : Envoie la sortie d'une commande comme entrée de la suivante (ex: `ls | grep .txt`).
* `&&` : Exécute la deuxième commande seulement si la première réussit.
* `||` : Exécute la deuxième commande seulement si la première échoue.
* `;` : Sépare plusieurs commandes exécutées séquentiellement.

---

## 3. Permissions 101

Sous Linux, chaque fichier possède des droits pour trois types d'utilisateurs : **Owner** (u), **Group** (g), et **Others** (o).

* **r** (read) : 4
* **w** (write) : 2
* **x** (execute) : 1

**Exemple :** `chmod 755 file`

* 7 (4+2+1) : Propriétaire a tous les droits.
* 5 (4+1) : Groupe et autres peuvent lire et exécuter.

---

## 4. Common Directories

La hiérarchie standard (FHS) :

* `/bin` & `/usr/bin` : Binaires des commandes utilisateur.
* `/etc` : Fichiers de configuration du système.
* `/home` : Répertoires personnels des utilisateurs.
* `/root` : Maison de l'administrateur (Super-utilisateur).
* `/tmp` : Fichiers temporaires (souvent effacés au redémarrage).
* `/var/log` : Fichiers de journaux (logs).
* `/dev` : Fichiers représentant les périphériques matériels.

---

## 5. Processes 101

Un processus est une instance d'un programme en cours d'exécution. Chaque processus a un **PID** (Process ID).

* `ps aux` : Liste tous les processus en cours.
* `top` ou `htop` : Affiche les processus en temps réel (utilisation CPU/RAM).
* `kill <PID>` : Arrête proprement un processus.
* `kill -9 <PID>` : Force l'arrêt immédiat.
* `bg` / `fg` : Envoie une tâche en arrière-plan ou la ramène au premier plan.

---

## 6. Automation

L'automatisation passe souvent par des scripts **Bash** et la planification de tâches.

* **Shebang** : La première ligne d'un script `#!/bin/bash` indique l'interpréteur à utiliser.
* **Crontab** : Utilisé pour planifier des tâches à intervalles réguliers.
* `crontab -e` : Éditer les tâches planifiées.
* Format : `min hour day month day_of_week command`



---

## 7. System Logs

Les logs sont essentiels pour le debug. Ils se trouvent majoritairement dans `/var/log`.

* `/var/log/syslog` (ou `messages`) : Logs généraux du système.
* `/var/log/auth.log` : Tentatives de connexion et authentification.
* `journalctl` : Outil pour consulter les logs gérés par *systemd*.
* `tail -f /var/log/syslog` : Suit l'ajout de nouvelles lignes en temps réel.

---

## 8. Package Management

Permet d'installer, mettre à jour et supprimer des logiciels.

| Distribution | Gestionnaire | Commandes courantes |
| --- | --- | --- |
| **Debian/Ubuntu** | `apt` | `apt update`, `apt install`, `apt remove` |
| **RHEL/CentOS** | `yum` / `dnf` | `dnf install`, `dnf upgrade` |
| **Arch Linux** | `pacman` | `pacman -S`, `pacman -Syu` |


