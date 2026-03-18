# Memento Linux Shell

## 1. Gestion des fichiers et des répertoires

| Commande | Description |
| --- | --- |
| `ls -l` | Affiche le contenu avec détails (droits, propriétaire, taille, date). |
| `ls -la` | Inclut les **fichiers cachés** dans l'affichage détaillé. |

### Recherche de fichier avec la commande `find`

La commande `find` permet de rechercher des fichiers et répertoires selon divers critères :

* **Par nom :**
* `find . -name "name.txt"` : Recherche exacte dans le répertoire courant (`.`).
* `find . -name "*.txt"` : Tous les fichiers se terminant par `.txt`.
* `find . -type f -name "config*"` : Uniquement les **fichiers** commençant par "config".
* `find . -type d -name "sauvegarde*"` : Uniquement les **répertoires** commençant par "sauvegarde".


* **Par taille :**
* `find / -size +100M` : Fichiers de plus de 100 Mo (recherche à la racine).
* `find . -size -10k` : Fichiers de moins de 10 Ko.


* **Par date :**
* `find . -mtime 2` : Modifié il y a exactement 2 jours.
* `find . -mtime -1` : Modifié il y a moins de 24 heures.
* `find . -mmin +10` : Modifié il y a plus de 10 minutes.


* **Avec exécution d'actions (`-exec`) :**
* `find /var/log -name "*.log" -mtime +30 -exec rm {} \;` : Supprime les logs de plus de 30 jours.
* `find . -type d -exec chmod 755 {} \;` : Change les permissions de tous les dossiers.



---

## 2. Recherche de mot dans le texte

### `grep` (Global Regular Expression Print)

* `grep "erreur" fichier.log` : Recherche le mot "erreur" dans le fichier.
* `grep -vE '^\s*#|^\s*$' fichier.txt` : Affiche les lignes **utiles** (exclut les commentaires `#` et les lignes vides).
* **Contexte de recherche :**
* `grep -A 2` : Affiche les 2 lignes **Après** (After).
* `grep -B 2` : Affiche les 2 lignes **Avant** (Before).
* `grep -C 2` : Affiche les 2 lignes **Autour** (Context).



### `wc` (Word Count)

* `wc -l access.log` : Compte le nombre total de lignes dans le fichier.

---

## 3. Utilisateurs et permissions

* `whoami` : Affiche l'utilisateur actuel.
* **Changement d'utilisateur :**
	* `su user` : Change d'utilisateur.
	* `su -l user` : Change d'utilisateur en chargeant son **environnement complet** (PATH, home, etc.).
* `getent passwd <user>` : Affiche les infos de l'utilisateur (depuis `/etc/passwd` ou LDAP).
* `sudo userdel -r -f user1` : Supprime l'utilisateur `user1`, son home directory (`-r`) et force l'action (`-f`).

---

## 4. Processus et services

### Information sur les Processus et services
* `ps -aux` : Liste détaillée de tous les processus du système.
* `dmesg --follow` : Affiche les messages du noyau en temps réel (logs matériel).
* `systemctl start|stop|status|restart [service]` : Gère les services (ex: `systemctl status sshd`).

### Arrêt de processus

* `kill PID` : Arrête proprement un processus via son ID.
* `kill -9 PID` : Force l'arrêt immédiat (Signal SIGKILL).
* `killall nom_du_processus` : Tue tous les processus portant ce nom.
* `pkill motif` : Tue les processus correspondant à un motif (regex).

---

## 5. Réseau et connectivité
### Liste des commandes  réseau utiles

| Commande | Usage |
| --- | --- |
| `hostname` | Nom de la machine. |
| `ip a` | Adresses IP et état des interfaces. |
| `ip route` | Table de routage (passerelle par défaut). |
| `netstat -anp --inet` | Connexions réseau actives avec PID des processus. |
| `lsof -i` | Liste les fichiers/sockets ouverts par le réseau. |
| `ss -atunp` | Statistiques des sockets (T=TCP, U=UDP, N=Numérique, P=Processus). |

### Configuration (NetworkManager & Classique)

* `nmcli device` : Liste les périphériques réseaux.
* `nmcli connection` : Liste les profils de connexion.
* `sudo ifconfig <interface> up/down` : Active ou désactive une interface manuellement.

---

## 6. Transferts et paquets
### SSH / SCP

- **Se connecter à un serveur distant via SSH :**
  ```bash
  ssh utilisateur@adresse_ip
  ```
  
- **Transférer un fichier vers un serveur distant avec SCP :**
  ```bash
  scp chemin/vers/fichier utilisateur@adresse_ip:chemin/destination
  ```
  
- **Transférer un fichier depuis un serveur distant avec SCP :**
  ```bash
  scp utilisateur@adresse_ip:chemin/vers/fichier chemin/local/destination
  ```

- **Transférer un répertoire entier avec SCP :**
  ```bash
  scp -r chemin/vers/répertoire utilisateur@adresse_ip:chemin/destination
  ```

- **Utiliser une clé privée pour se connecter :**
  ```bash
  ssh -i chemin/vers/clé utilisateur@adresse_ip
  ```

### Gestion des dépôts (Ubuntu/Debian)

- **Mettre à jour la liste des paquets :**
  ```bash
  sudo apt update
  sudo dnf check-update
  ```

- **Mettre à niveau les paquets installés :**
  ```bash
  sudo apt upgrade
  sudo dnf upgrade
  ```

- **Installer un paquet :**
  ```bash
  sudo apt install nom_du_paquet
  sudo dnf install nom_du_paquet
  ```

- **Désinstaller un paquet :**
  ```bash
  sudo apt remove nom_du_paquet
  sudo dnf remone nom_du_paquet
  ```
  
- **Ajouter un dépôt :**
  1. Ouvrir le fichier sources :
     ```bash
    sudo nano /etc/apt/sources.list
     
    (Créer un fichier `.repo` dans `/etc/yum.repos.d/`, par exemple :)
     sudo nano /etc/yum.repos.d/mon_dépôt.repo
	    [nom_du_dépôt]
		name=Description du dépôt
		baseurl=http://url_du_dépôt
		enabled=1
		gpgcheck=1

     ```
  2. Ajouter la ligne correspondant au dépôt (ex: `deb http://archive.ubuntu.com/ubuntu/ focal main`).

- **Installer un paquet à partir d’un dépôt spécifique :**
  ```bash
  sudo apt install nom_du_paquet -t nom_du_dépôt
  ```

- **Supprimer un dépôt :**
  1. Ouvrir le fichier sources :
     ```bash
     sudo nano /etc/apt/sources.list
     ```
  2. Commenter ou supprimer la ligne du dépôt.

- **Vérifier les dépôts activés :**
  ```bash
  apt policy
  dnf repolist
  ```

---

## 7. Structure du répertoire `/usr/local`


- **`/usr/local/`**
  - Contient les fichiers utilisés par le système local mais qui ne sont pas gérés par le gestionnaire de paquets.
### Sous-répertoires Communs

| Sous-répertoire          | Description                                               |
|-------------------------|-----------------------------------------------------------|
| **`bin/`                | Contient les exécutables locaux.                          |
| **`etc/`                | Contient les fichiers de configuration locale.            |
| **`include/`            | Contient les fichiers d'en-tête pour les bibliothèques.  |
| **`lib/`                | Contient les bibliothèques partagées et locales.         |
| **`man/`                | Contient les pages de manuel locales.                     |
| **`sbin/`               | Contient les exécutables systèmes locaux (pour l'administration). |
| **`share/`              | Contient les fichiers de données partagés (ex: documentation, icônes). |
| **`src/`                | Contient le code source des programmes installés localement. |

### Points Clés

- **Isolation :** Le répertoire `/usr/local` est utilisé pour séparer les applications installées manuellement des applications gérées par le système et son gestionnaire de paquets.
- **Privilèges :** Les utilisateurs normaux n'ont généralement pas besoin de modifier ce répertoire, mais l'accès est souvent requis pour les administrateurs.
- **Personnalisation :** Les sous-répertoires permettent de garder une organisation claire pour les diverses ressources et configurations associées aux programmes installés localement.