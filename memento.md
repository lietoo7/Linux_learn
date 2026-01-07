# 🐧 Memento Linux Shell

## 1. Gestion des fichiers et des répertoires

| Commande | Description |
| --- | --- |
| `ls -l` | Affiche le contenu avec détails (droits, propriétaire, taille, date). |
| `ls -la` | Inclut les **fichiers cachés** dans l'affichage détaillé. |

### 🔍 Utilisation de `find`

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

## 2. Recherche et traitement de texte

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

* `ps -aux` : Liste détaillée de tous les processus du système.
* `dmesg --follow` : Affiche les messages du noyau en temps réel (logs matériel).
* `systemctl start|stop|status|restart [service]` : Gère les services (ex: `systemctl status sshd`).

### 🛑 Arrêt de processus

* `kill PID` : Arrête proprement un processus via son ID.
* `kill -9 PID` : Force l'arrêt immédiat (Signal SIGKILL).
* `killall nom_du_processus` : Tue tous les processus portant ce nom.
* `pkill motif` : Tue les processus correspondant à un motif (regex).

---

## 5. Réseau et connectivité

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

* **SSH / SCP :**
* `scp ./fichier user@ip:/destination` : Copie sécurisée vers un hôte distant.


* **Gestion des dépôts (Ubuntu/Debian) :**
* `add-apt-repository --remove ppa:nom/ppa` : Supprime un dépôt PPA.



---

## 7. Structure du répertoire `/usr/local`

> **Note :** Ce répertoire est destiné aux logiciels installés manuellement (souvent compilés), en dehors du gestionnaire de paquets officiel (`apt`, `dnf`). Cela évite que les mises à jour du système n'écrasent vos programmes personnalisés.

* `/usr/local/bin` : Exécutables pour les utilisateurs.
* `/usr/local/sbin` : Exécutables pour l'administration.
* `/usr/local/lib` : Bibliothèques partagées.
 
