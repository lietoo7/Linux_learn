# Fiche 7 : System Logs

Les logs (journaux) sont la mémoire du système. Ils enregistrent les événements liés au noyau, aux services, aux erreurs et aux connexions des utilisateurs.

## 1. Où se trouvent les logs ?

La quasi-totalité des fichiers journaux se trouve dans le répertoire **/var/log**.

* `/var/log/syslog` (ou `/var/log/messages` sur RHEL) : Journal global du système.
* `/var/log/auth.log` (ou `secure`) : Journal des tentatives de connexion et d'utilisation de `sudo`.
* `/var/log/kern.log` : Messages spécifiques au noyau (kernel).
* `/var/log/dmesg` : Messages liés au démarrage du système et au matériel.
* `/var/log/apache2/` ou `/var/log/nginx/` : Logs des serveurs web (si installés).

---

## 2. Consulter les logs avec Systemd : `journalctl`

Sur les distributions modernes (Ubuntu, Debian, CentOS 7+), les logs sont gérés par un service appelé **journald**. On utilise la commande `journalctl` pour les lire.

* `journalctl` : Affiche l'intégralité des logs (très long).
* `journalctl -u ssh` : Affiche les logs d'un service spécifique (ici SSH).
* `journalctl -f` : Affiche les nouveaux logs en temps réel (mode "follow").
* `journalctl -p err` : Affiche uniquement les erreurs et les messages plus graves.
* `journalctl --since "1 hour ago"` : Filtre par date/heure.

---

## 3. Commandes classiques pour lire les fichiers logs

Comme les fichiers dans `/var/log` sont souvent volumineux, on utilise des commandes spécifiques pour ne pas saturer l'écran :

* `tail -n 20 /var/log/syslog` : Affiche les 20 **dernières** lignes.
* `tail -f /var/log/auth.log` : Surveille en direct les tentatives de connexion (très utile pour le debug).
* `less /var/log/syslog` : Permet de naviguer dans le fichier (flèches, recherche avec `/`).
* `grep "Failed password" /var/log/auth.log` : Filtre les tentatives de connexion échouées.

---

## 4. Rotation des logs : `logrotate`

Les fichiers logs pourraient remplir tout votre disque dur s'ils n'étaient pas gérés.

* **Logrotate** est l'outil qui archive périodiquement les vieux fichiers logs (ex: `syslog.1`, `syslog.2.gz`) et supprime les plus anciens après un certain temps.
* Sa configuration se trouve dans `/etc/logrotate.conf`.

---

## Résumé pratique

> **"Au secours, mon service ne démarre pas !"**
> 1. Tente de démarrer : `sudo systemctl start mon_service`
> 2. Regarde l'erreur immédiate : `systemctl status mon_service`
> 3. Fouille les logs pour le détail : `journalctl -u mon_service -n 50`
 
