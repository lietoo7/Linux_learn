# 12 — Systèmes de Sauvegarde Linux

> **Public cible :** Administrateurs systèmes Linux (niveau intermédiaire à avancé)  
> **Familles couvertes :** Debian/Ubuntu/Mint · RHEL/CentOS/AlmaLinux · Arch Linux  
> **Dernière révision :** 2026

---

## Table des matières

1. [Introduction : Philosophie de la sauvegarde](#1-introduction--philosophie-de-la-sauvegarde)
2. [Synchronisation de fichiers avec rsync](#2-synchronisation-de-fichiers-avec-rsync)
3. [Archivage avec tar](#3-archivage-avec-tar)
4. [Outils de backup réseau](#4-outils-de-backup-réseau)
5. [Backup chiffré](#5-backup-chiffré)
6. [Instantanés système (Snapshots)](#6-instantanés-système-snapshots)
7. [Tableau comparatif des commandes](#7-tableau-comparatif-des-commandes)

---

## 1. Introduction : Philosophie de la sauvegarde

### 1.1 La stratégie 3-2-1

La **stratégie 3-2-1** est le standard de référence en matière de sauvegarde. Elle repose sur trois principes fondamentaux :

| Principe | Signification |
|----------|---------------|
| **3** copies des données | L'original + 2 sauvegardes distinctes |
| **2** supports différents | Ex : disque local + NAS, ou disque + bande |
| **1** copie hors site | Cloud, datacenter distant, support physique externalisé |

Cette stratégie protège contre les pannes matérielles, les catastrophes locales (incendie, inondation) et les erreurs humaines.

### 1.2 Types de sauvegardes

```
Complète (Full)      → Copie intégrale de toutes les données sélectionnées.
                        Avantage : restauration simple. Inconvénient : long et volumineux.

Incrémentale         → Seules les données modifiées depuis la DERNIÈRE sauvegarde
(Incremental)          (complète ou incrémentale) sont copiées.
                        Avantage : rapide et léger. Inconvénient : restauration en cascade.

Différentielle       → Seules les données modifiées depuis la DERNIÈRE sauvegarde
(Differential)         COMPLÈTE sont copiées.
                        Compromis entre les deux approches.
```

### 1.3 L'immuabilité des sauvegardes

L'immuabilité (`immutability`) est un principe critique : une sauvegarde immuable **ne peut pas être modifiée, chiffrée ou supprimée** pendant une durée définie, même par un administrateur root compromis.

Objectifs :
- Protection contre les **ransomwares** (chiffrement des backups)
- Protection contre les **suppressions accidentelles** ou malveillantes
- Conformité réglementaire (RGPD, PCI-DSS, HIPAA)

Solutions techniques :
- **Append-only mode** de BorgBackup / Restic
- **Object Lock** S3 (stockage cloud)
- **Snapshots ZFS/Btrfs** en lecture seule
- Sauvegardes vers des médias WORM (*Write Once, Read Many*)

> ⚠️ **Règle d'or :** Une sauvegarde non testée est une sauvegarde qui n'existe pas. Planifiez des tests de restauration réguliers.

---

## 2. Synchronisation de fichiers avec rsync

### 2.1 Présentation

`rsync` (Remote Sync) est l'outil de synchronisation de référence sous Linux. Il utilise un **algorithme delta** : seules les parties modifiées des fichiers sont transférées, minimisant la bande passante et le temps d'exécution.

### 2.2 Syntaxe de base

```bash
rsync [OPTIONS] SOURCE DESTINATION
```

> ⚠️ **Attention au slash final (`/`) sur la source :**
> - `rsync -av /data/docs /backup/`  → copie le répertoire `docs/` dans `/backup/docs/`
> - `rsync -av /data/docs/ /backup/` → copie le **contenu** de `docs/` directement dans `/backup/`

### 2.3 Options essentielles

| Option | Signification |
|--------|---------------|
| `-a` / `--archive` | Mode archive : préserve permissions, timestamps, liens symboliques, propriétaire, groupe |
| `-v` / `--verbose` | Affiche les fichiers traités |
| `-z` / `--compress` | Compression des données en transit (utile sur liaisons lentes) |
| `-P` | Combine `--progress` + `--partial` (reprend les transferts interrompus) |
| `--delete` | Supprime en destination les fichiers absents de la source (miroir exact) |
| `--exclude=PATTERN` | Exclut les fichiers correspondant au motif |
| `--exclude-from=FILE` | Lit les exclusions depuis un fichier |
| `-n` / `--dry-run` | Simulation sans modification réelle (toujours tester avant d'exécuter) |
| `--bwlimit=KBPS` | Limite la bande passante (ex : `--bwlimit=50000` pour 50 MB/s) |
| `--checksum` | Vérifie par somme de contrôle plutôt que par date/taille |
| `--link-dest=DIR` | Sauvegardes incrémentales avec hardlinks (voir §2.6) |

### 2.4 Exemples pratiques

#### Synchronisation locale

```bash
# Synchronisation simple avec préservation des métadonnées
rsync -av /var/www/html/ /backup/www/

# Miroir exact (suppression des fichiers absents de la source)
rsync -av --delete /home/edouard/ /backup/home/edouard/

# Simulation préalable (bonne pratique)
rsync -avn --delete /home/edouard/ /backup/home/edouard/
```

#### Synchronisation distante via SSH

```bash
# Sauvegarde locale → serveur distant
rsync -avz -e "ssh -p 2222" /data/production/ user@192.168.1.100:/backup/production/

# Récupération depuis un serveur distant (restauration)
rsync -avz user@192.168.1.100:/backup/production/ /data/production/

# Avec limitation de bande passante et reprise automatique
rsync -avzP --bwlimit=20000 -e ssh /data/ user@backup-server:/remote/data/
```

#### Exclusions avancées

```bash
# Exclusion par motif inline
rsync -av --exclude='*.log' --exclude='*.tmp' --exclude='.cache/' /home/ /backup/home/

# Exclusion via fichier (plus lisible pour les listes complexes)
cat > /etc/rsync-excludes.txt << 'EOF'
*.log
*.tmp
*.swap
.cache/
.mozilla/
node_modules/
__pycache__/
EOF

rsync -av --exclude-from=/etc/rsync-excludes.txt /home/ /backup/home/
```

### 2.5 Options de performance

```bash
# Transfert rapide sur réseau local (désactive compression, active checksum)
rsync -a --no-compress --inplace /data/ /backup/

# Optimisation pour de nombreux petits fichiers
rsync -a --whole-file /data/ /backup/

# Parallélisation avec GNU Parallel (pour de très grands volumes)
find /data -maxdepth 1 -type d | parallel -j4 rsync -a {} /backup/
```

### 2.6 Sauvegardes incrémentales avec hardlinks (`--link-dest`)

Cette technique crée des **sauvegardes journalières incomplètes en apparence**, mais chaque dossier de sauvegarde contient en réalité l'image complète du système via les hardlinks. L'espace disque n'est consommé que pour les fichiers réellement modifiés.

```bash
#!/bin/bash
# /usr/local/sbin/rsync-incremental.sh

SOURCE="/home/"
DEST="/backup/incremental"
DATE=$(date +%Y-%m-%d)
LAST_BACKUP=$(ls -1d ${DEST}/[0-9]* 2>/dev/null | tail -1)

mkdir -p "${DEST}/${DATE}"

if [ -n "${LAST_BACKUP}" ]; then
    rsync -av --delete \
        --link-dest="${LAST_BACKUP}" \
        "${SOURCE}" "${DEST}/${DATE}/"
    echo "Sauvegarde incrémentale vers ${DEST}/${DATE} (référence : ${LAST_BACKUP})"
else
    rsync -av "${SOURCE}" "${DEST}/${DATE}/"
    echo "Première sauvegarde complète vers ${DEST}/${DATE}"
fi
```

### 2.7 Planification avec cron

```bash
# Éditer la crontab root
crontab -e

# Sauvegarde quotidienne à 02h00
0 2 * * * /usr/local/sbin/rsync-incremental.sh >> /var/log/rsync-backup.log 2>&1

# Sauvegarde hebdomadaire complète le dimanche à 03h00
0 3 * * 0 rsync -av --delete /data/ /backup/weekly/ >> /var/log/rsync-weekly.log 2>&1
```

---

## 3. Archivage avec tar

### 3.1 Présentation

`tar` (Tape ARchive) est l'outil standard de création d'archives sous Linux. Il **ne compresse pas nativement** mais délègue cette fonction à des outils externes (`gzip`, `bzip2`, `xz`, `zstd`).

### 3.2 Syntaxe générale

```bash
tar [MODE] [OPTIONS] [ARCHIVE] [FICHIERS/DOSSIERS]
```

**Modes principaux :**

| Option | Mode |
|--------|------|
| `-c` | Créer (Create) |
| `-x` | Extraire (eXtract) |
| `-t` | Lister (lisTe) |
| `-r` | Ajouter à une archive existante (non compressée) |
| `-u` | Mettre à jour (Update) |

### 3.3 Compression : comparaison des algorithmes

| Algorithme | Flag tar | Extension | Vitesse | Ratio | Usage recommandé |
|------------|----------|-----------|---------|-------|-----------------|
| gzip | `-z` | `.tar.gz` / `.tgz` | ★★★★☆ Rapide | ★★★☆☆ Moyen | Usage général, compatibilité maximale |
| bzip2 | `-j` | `.tar.bz2` | ★★☆☆☆ Lent | ★★★★☆ Bon | Archives peu fréquentes, stockage longue durée |
| xz | `-J` | `.tar.xz` | ★☆☆☆☆ Très lent | ★★★★★ Excellent | Paquets logiciels, stockage longue durée |
| zstd | `--zstd` | `.tar.zst` | ★★★★★ Très rapide | ★★★★☆ Bon | Backups fréquents, usage moderne recommandé |

### 3.4 Création d'archives

```bash
# Archive gzip (usage courant)
tar -czf /backup/etc-$(date +%Y%m%d).tar.gz /etc/

# Archive bzip2 (meilleur ratio, plus lent)
tar -cjf /backup/home-$(date +%Y%m%d).tar.bz2 /home/

# Archive xz (ratio maximal)
tar -cJf /backup/srv-$(date +%Y%m%d).tar.xz /srv/

# Archive zstd avec niveau de compression (recommandé pour les backups réguliers)
tar --zstd -cf /backup/var-$(date +%Y%m%d).tar.zst /var/

# Compression multi-thread xz (utilise tous les cœurs disponibles)
XZ_OPT="-T0" tar -cJf /backup/data-$(date +%Y%m%d).tar.xz /data/

# Avec barre de progression (via pv)
tar -czf - /home/ | pv -s $(du -sb /home/ | awk '{print $1}') > /backup/home.tar.gz
```

### 3.5 Extraction d'archives

```bash
# Extraire dans le répertoire courant
tar -xzf /backup/etc-20260310.tar.gz

# Extraire dans un répertoire cible spécifique
tar -xzf /backup/etc-20260310.tar.gz -C /tmp/restore/

# Extraire un seul fichier depuis une archive
tar -xzf /backup/etc-20260310.tar.gz etc/nginx/nginx.conf -C /tmp/

# Extraire avec préservation des permissions et propriétaires (nécessite root)
tar -xzpf /backup/etc-20260310.tar.gz -C /
```

### 3.6 Listage et vérification

```bash
# Lister le contenu d'une archive
tar -tzf /backup/etc-20260310.tar.gz

# Lister avec détails (permissions, propriétaire, taille)
tar -tvzf /backup/etc-20260310.tar.gz

# Vérifier l'intégrité (compare archive vs système de fichiers)
tar -czf - /etc/ | md5sum
tar -tzf /backup/etc-20260310.tar.gz > /dev/null && echo "Archive OK"
```

### 3.7 Gestion des permissions et propriétaires

```bash
# Préserver les permissions, ACL et attributs étendus (recommandé pour /etc, /home)
tar --preserve-permissions --same-owner -czf /backup/system.tar.gz /etc/ /home/

# Exclure des fichiers/répertoires
tar -czf /backup/home.tar.gz \
    --exclude='/home/*/.cache' \
    --exclude='/home/*/.local/share/Trash' \
    --exclude='*.log' \
    /home/

# Sauvegarder en préservant les liens symboliques et les fichiers spéciaux
tar -czpf /backup/full.tar.gz \
    --acls \
    --xattrs \
    --selinux \
    /etc/ /var/ /usr/local/

# Archive avec vérification immédiate
tar -czf /backup/data.tar.gz /data/ && tar -tzf /backup/data.tar.gz > /dev/null \
    && echo "✓ Archive créée et vérifiée" || echo "✗ Erreur lors de l'archivage"
```

### 3.8 Calcul et vérification des checksums

```bash
# Générer un checksum SHA256 de l'archive
sha256sum /backup/etc-20260310.tar.gz > /backup/etc-20260310.tar.gz.sha256

# Vérifier ultérieurement
sha256sum -c /backup/etc-20260310.tar.gz.sha256

# Script de sauvegarde tar avec checksum automatique
#!/bin/bash
ARCHIVE="/backup/etc-$(date +%Y%m%d-%H%M).tar.gz"
tar -czf "${ARCHIVE}" /etc/ 2>/dev/null
sha256sum "${ARCHIVE}" > "${ARCHIVE}.sha256"
echo "Archive : ${ARCHIVE}"
echo "Checksum : $(cat ${ARCHIVE}.sha256)"
```

---

## 4. Outils de backup réseau

### 4.1 Restic

#### Présentation

**Restic** est un outil de sauvegarde moderne, rapide et sécurisé. Ses caractéristiques clés :
- **Déduplication au niveau des blocs** (économie d'espace significative)
- **Chiffrement AES-256 natif** (toujours actif, non optionnel)
- **Backends multiples** : local, SFTP, S3, Azure, GCS, REST, Backblaze B2
- Architecture **client-only** (pas de serveur dédié requis)

#### Installation

```bash
# Debian/Ubuntu/Mint
sudo apt install restic

# RHEL/CentOS/AlmaLinux (via EPEL)
sudo dnf install epel-release
sudo dnf install restic

# Arch Linux
sudo pacman -S restic

# Mise à jour auto (toutes distributions)
sudo restic self-update
```

#### Initialisation d'un dépôt

```bash
# Dépôt local
restic init --repo /backup/restic-repo

# Dépôt SFTP sur serveur distant
restic init --repo sftp:user@192.168.1.100:/backup/restic-repo

# Dépôt S3 (compatible Minio, AWS S3)
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
restic init --repo s3:http://minio.local:9000/bucket-name
```

#### Opérations courantes

```bash
# Sauvegarde
restic backup --repo /backup/restic-repo /home/ /etc/

# Sauvegarde avec exclusions
restic backup --repo /backup/restic-repo \
    --exclude='*.log' \
    --exclude='/home/*/.cache' \
    /home/ /etc/

# Sauvegarde avec tag pour organisation
restic backup --repo /backup/restic-repo --tag "production,daily" /data/

# Lister les snapshots
restic snapshots --repo /backup/restic-repo

# Restauration complète d'un snapshot
restic restore --repo /backup/restic-repo latest --target /tmp/restore/

# Restauration d'un fichier spécifique
restic restore --repo /backup/restic-repo latest \
    --include /etc/nginx/nginx.conf \
    --target /tmp/restore/

# Navigation interactive dans les snapshots (mount FUSE)
restic mount --repo /backup/restic-repo /mnt/restic-snapshots
# Puis : ls /mnt/restic-snapshots/snapshots/

# Vérification de l'intégrité du dépôt
restic check --repo /backup/restic-repo

# Politique de rétention (pruning)
restic forget --repo /backup/restic-repo \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 6 \
    --keep-yearly 2 \
    --prune
```

#### Script de sauvegarde Restic automatisé

```bash
#!/bin/bash
# /usr/local/sbin/restic-backup.sh

export RESTIC_REPOSITORY="/backup/restic-repo"
export RESTIC_PASSWORD_FILE="/etc/restic/password"

LOG="/var/log/restic-backup.log"
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

echo "=== Début sauvegarde : ${TIMESTAMP} ===" >> "${LOG}"

# Sauvegarde
restic backup \
    --exclude-file=/etc/restic/excludes.txt \
    --tag "$(hostname),daily" \
    /home/ /etc/ /var/www/ >> "${LOG}" 2>&1

if [ $? -eq 0 ]; then
    echo "✓ Sauvegarde réussie" >> "${LOG}"
else
    echo "✗ ERREUR lors de la sauvegarde" >> "${LOG}"
    exit 1
fi

# Politique de rétention
restic forget \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 3 \
    --prune >> "${LOG}" 2>&1

# Vérification hebdomadaire (le dimanche)
if [ "$(date +%u)" = "7" ]; then
    restic check >> "${LOG}" 2>&1
fi

echo "=== Fin sauvegarde : $(date +"%Y-%m-%d %H:%M:%S") ===" >> "${LOG}"
```

### 4.2 Bacula

#### Présentation

**Bacula** est une solution de backup réseau **client/serveur** de niveau entreprise. Architecture en composants distincts :

```
┌─────────────────────────────────────────────────────┐
│                  Réseau de sauvegarde                │
│                                                     │
│  ┌──────────────┐    ┌──────────────────────────┐   │
│  │  Bacula-Dir  │◄──►│  Bacula-SD (Storage)     │   │
│  │  (Director)  │    │  /dev/tape0 ou /backup/  │   │
│  └──────┬───────┘    └──────────────────────────┘   │
│         │                                           │
│         ▼                                           │
│  ┌──────────────┐    ┌──────────────┐               │
│  │  Bacula-FD   │    │  Bacula-FD   │  (Clients)    │
│  │  serveur-web │    │  db-server   │               │
│  └──────────────┘    └──────────────┘               │
└─────────────────────────────────────────────────────┘
```

| Composant | Rôle |
|-----------|------|
| **Director** | Cerveau du système, orchestre toutes les opérations |
| **Storage Daemon (SD)** | Gère les médias de stockage (disques, bandes) |
| **File Daemon (FD)** | Agent installé sur chaque client à sauvegarder |
| **Catalog** | Base de données (MySQL/PostgreSQL) des métadonnées |
| **Console (bconsole)** | Interface d'administration en ligne de commande |

#### Installation sur serveur (Debian/Ubuntu)

```bash
# Serveur Bacula complet
sudo apt install bacula bacula-console

# Seulement le File Daemon (clients)
sudo apt install bacula-fd
```

#### Installation sur serveur (RHEL/AlmaLinux)

```bash
sudo dnf install epel-release
sudo dnf install bacula-director bacula-storage bacula-console bacula-client
```

#### Installation sur serveur (Arch Linux)

```bash
sudo pacman -S bacula
# ou via AUR
yay -S bacula-server bacula-client
```

#### Configuration minimale du Director (`/etc/bacula/bacula-dir.conf`)

```bash
# Définition du Director
Director {
  Name = bacula-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/spool/bacula"
  PidDirectory = "/var/run"
  Maximum Concurrent Jobs = 5
  Password = "MotDePasseDirecteur"
  Messages = Daemon
}

# Client à sauvegarder
Client {
  Name = serveur-web-fd
  Address = 192.168.1.10
  FDPort = 9102
  Catalog = MyCatalog
  Password = "MotDePasseClient"
  File Retention = 60 days
  Job Retention = 6 months
  AutoPrune = yes
}

# Définition d'un Job de sauvegarde
Job {
  Name = "Backup-ServeurWeb"
  Type = Backup
  Level = Incremental
  Client = serveur-web-fd
  FileSet = "Full-Set"
  Schedule = "WeeklyCycle"
  Storage = File-Storage
  Messages = Standard
  Pool = File
  Priority = 10
  Write Bootstrap = "/var/spool/bacula/%c.bsr"
}

# Fileset : ce qui est sauvegardé
FileSet {
  Name = "Full-Set"
  Include {
    Options {
      signature = MD5
      compression = GZIP
    }
    File = /home
    File = /etc
    File = /var/www
  }
  Exclude {
    File = /var/lib/bacula
    File = /proc
    File = /tmp
    File = /sys
  }
}

# Schedule de sauvegarde
Schedule {
  Name = "WeeklyCycle"
  Run = Full 1st sun at 23:05
  Run = Differential 2nd-5th sun at 23:05
  Run = Incremental mon-sat at 23:05
}
```

#### Commandes bconsole essentielles

```bash
# Lancer la console Bacula
sudo bconsole

# Dans bconsole :
*status director          # État du Director
*status client            # État des clients
*list jobs                # Lister les jobs
*list files jobid=42      # Lister les fichiers d'un job
*restore                  # Assistant de restauration interactif
*run job="Backup-ServeurWeb"  # Lancer un job manuellement
*cancel jobid=42          # Annuler un job en cours
```

---

## 5. Backup chiffré

### 5.1 BorgBackup

#### Présentation

**BorgBackup** (Borg) est un outil de sauvegarde avec **déduplication, compression et chiffrement intégrés**. Il est particulièrement efficace pour les sauvegardes locales et distantes via SSH.

Caractéristiques :
- Chiffrement **AES-256-CTR** + authentification **HMAC-SHA256**
- Déduplication au niveau des chunks (blocs de données variables)
- Compression : lz4, zstd, zlib, lzma
- Support des modes **append-only** (immuabilité)

#### Installation

```bash
# Debian/Ubuntu/Mint
sudo apt install borgbackup

# RHEL/CentOS/AlmaLinux
sudo dnf install epel-release
sudo dnf install borgbackup

# Arch Linux
sudo pacman -S borg
```

#### Initialisation et gestion des clés

```bash
# Initialisation d'un dépôt local avec chiffrement (recommandé)
borg init --encryption=repokey-blake2 /backup/borg-repo

# Initialisation sur serveur distant (SSH)
borg init --encryption=repokey-blake2 user@backup-server:/backup/borg-repo

# IMPORTANT : Exporter et sauvegarder la clé de chiffrement
borg key export /backup/borg-repo /root/borg-key-backup.txt
# Stocker ce fichier EN DEHORS du dépôt (coffre-fort, stockage cloud séparé)

# Modes de chiffrement disponibles :
# - repokey       : clé stockée dans le dépôt (protégée par passphrase)
# - keyfile       : clé stockée dans ~/.config/borg/keys/ (plus sécurisé)
# - repokey-blake2: comme repokey mais avec BLAKE2 (plus rapide)
# - authenticated : intégrité sans confidentialité
# - none          : pas de chiffrement (déconseillé)
```

#### Opérations courantes

```bash
# Variable d'environnement pour éviter la saisie répétée de la passphrase
export BORG_PASSPHRASE="VotrePassphraseSécurisée"

# Création d'une sauvegarde
borg create \
    --stats \
    --compression lz4 \
    /backup/borg-repo::"{hostname}-{now:%Y-%m-%d}" \
    /home/ /etc/ /var/www/ \
    --exclude '/home/*/.cache' \
    --exclude '*.tmp'

# Lister les archives du dépôt
borg list /backup/borg-repo

# Informations détaillées sur une archive
borg info /backup/borg-repo::serveur-2026-03-15

# Lister les fichiers dans une archive
borg list /backup/borg-repo::serveur-2026-03-15

# Restauration complète
borg extract /backup/borg-repo::serveur-2026-03-15 --target /tmp/restore/

# Restauration d'un fichier spécifique (chemin relatif sans slash initial)
borg extract /backup/borg-repo::serveur-2026-03-15 home/edouard/.bashrc

# Montage FUSE pour navigation/restauration sélective
borg mount /backup/borg-repo /mnt/borg-backup
ls /mnt/borg-backup/
borg umount /mnt/borg-backup

# Politique de rétention
borg prune /backup/borg-repo \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 6 \
    --keep-yearly 2 \
    --list

# Vérification de l'intégrité
borg check /backup/borg-repo
```

#### Mode append-only (immuabilité)

```bash
# Sur le serveur de backup distant : créer un utilisateur dédié avec mode append-only
# Ajouter dans ~/.ssh/authorized_keys du serveur :
command="borg serve --append-only --restrict-to-path /backup/borg-repo",restrict ssh-rsa AAAA...

# Ainsi, même si un client est compromis, il ne peut PAS supprimer d'archives
```

#### Script de sauvegarde Borg complet

```bash
#!/bin/bash
# /usr/local/sbin/borg-backup.sh

export BORG_REPO="/backup/borg-repo"
export BORG_PASSPHRASE="$(cat /etc/borg/passphrase)"

LOG="/var/log/borg-backup.log"
HOSTNAME=$(hostname -s)

info() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] INFO: $*" >> "${LOG}"; }
error() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $*" >> "${LOG}"; }

info "=== Début sauvegarde Borg ==="

# Création de l'archive
borg create \
    --stats \
    --compression lz4 \
    "${BORG_REPO}::${HOSTNAME}-{now:%Y-%m-%dT%H:%M:%S}" \
    /home/ /etc/ /var/www/ /opt/ \
    --exclude '/home/*/.cache' \
    --exclude '/home/*/.local/share/Trash' \
    --exclude '*.log' \
    >> "${LOG}" 2>&1

BACKUP_EXIT=$?

# Pruning
borg prune \
    --list \
    --prefix "${HOSTNAME}-" \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 3 \
    "${BORG_REPO}" >> "${LOG}" 2>&1

PRUNE_EXIT=$?

if [ ${BACKUP_EXIT} -eq 0 ] && [ ${PRUNE_EXIT} -eq 0 ]; then
    info "✓ Sauvegarde et pruning réussis"
    exit 0
else
    error "✗ Erreur (backup: ${BACKUP_EXIT}, prune: ${PRUNE_EXIT})"
    exit 1
fi
```

### 5.2 Duplicati

**Duplicati** est une solution de backup graphique (GUI web) avec chiffrement AES-256, orientée utilisateur final. Elle supporte de nombreux backends (S3, Google Drive, OneDrive, FTP, SSH, etc.).

#### Installation

```bash
# Debian/Ubuntu/Mint
wget https://updates.duplicati.com/beta/duplicati_2.x.x_all.deb
sudo apt install ./duplicati_2.x.x_all.deb
sudo systemctl enable --now duplicati

# RHEL/AlmaLinux (via RPM direct)
sudo dnf install mono-core
wget https://updates.duplicati.com/beta/duplicati-2.x.x-1.noarch.rpm
sudo rpm -i duplicati-2.x.x-1.noarch.rpm

# Arch Linux (AUR)
yay -S duplicati
sudo systemctl enable --now duplicati
```

#### Accès à l'interface

```bash
# Interface web accessible à :
# http://localhost:8200

# Pour accès distant sécurisé via tunnel SSH :
ssh -L 8200:localhost:8200 user@serveur-distant
# Puis ouvrir http://localhost:8200 en local
```

### 5.3 Bonnes pratiques de gestion des clés de chiffrement

```
┌─────────────────────────────────────────────────────────────────┐
│                 RÈGLES DE GESTION DES CLÉS                      │
├─────────────────────────────────────────────────────────────────┤
│ 1. JAMAIS stocker la passphrase dans le même endroit que        │
│    les sauvegardes chiffrées                                    │
│                                                                 │
│ 2. Exporter et sauvegarder la clé de chiffrement :             │
│    - Coffre-fort physique (imprimé)                             │
│    - Gestionnaire de mots de passe d'entreprise                 │
│    - Stockage cloud chiffré SÉPARÉ                              │
│                                                                 │
│ 3. Tester la restauration AVEC chiffrement périodiquement       │
│                                                                 │
│ 4. Documenter la procédure de récupération des clés             │
│                                                                 │
│ 5. Rotation des clés selon la politique de sécurité             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Instantanés système (Snapshots)

### 6.1 LVM Snapshots

#### Prérequis

LVM (Logical Volume Manager) est disponible sur toutes les distributions. Les snapshots LVM capturent l'état d'un volume logique à un instant T.

```bash
# Vérifier la configuration LVM actuelle
sudo pvs   # Physical Volumes
sudo vgs   # Volume Groups
sudo lvs   # Logical Volumes
```

#### Création et utilisation

```bash
# Créer un snapshot de /dev/vg0/data (nécessite de l'espace libre dans le VG)
sudo lvcreate -L 10G -s -n data-snapshot /dev/vg0/data

# Lister les snapshots
sudo lvs -a

# Monter le snapshot pour consultation/restauration
sudo mkdir -p /mnt/snapshot
sudo mount -o ro /dev/vg0/data-snapshot /mnt/snapshot
ls /mnt/snapshot/

# Copier des fichiers depuis le snapshot
cp /mnt/snapshot/etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak

# Démonter et supprimer le snapshot
sudo umount /mnt/snapshot
sudo lvremove /dev/vg0/data-snapshot

# Restauration complète (ATTENTION : destructif, écrase le volume source)
sudo lvconvert --merge /dev/vg0/data-snapshot
# Nécessite un redémarrage si le volume est actif
```

#### Automatisation des snapshots LVM

```bash
#!/bin/bash
# /usr/local/sbin/lvm-snapshot.sh

VG="vg0"
LV="data"
SNAP_NAME="${LV}-snap-$(date +%Y%m%d-%H%M)"
SNAP_SIZE="10G"
MOUNT_POINT="/mnt/lvm-snapshot"
BACKUP_DEST="/backup/lvm-daily"

# Création du snapshot
lvcreate -L ${SNAP_SIZE} -s -n "${SNAP_NAME}" "/dev/${VG}/${LV}"

# Montage en lecture seule
mkdir -p "${MOUNT_POINT}"
mount -o ro "/dev/${VG}/${SNAP_NAME}" "${MOUNT_POINT}"

# Sauvegarde depuis le snapshot (consistant)
rsync -av "${MOUNT_POINT}/" "${BACKUP_DEST}/"

# Nettoyage
umount "${MOUNT_POINT}"
lvremove -f "/dev/${VG}/${SNAP_NAME}"

echo "Snapshot ${SNAP_NAME} créé, sauvegardé et supprimé."
```

### 6.2 Btrfs Snapshots

#### Présentation

**Btrfs** (B-Tree File System) intègre nativement les snapshots avec **Copy-on-Write (CoW)**. Les snapshots sont quasi-instantanés et ne consomment initialement aucun espace supplémentaire.

```bash
# Vérifier si un volume est Btrfs
df -T /
findmnt -t btrfs

# Lister les sous-volumes Btrfs
sudo btrfs subvolume list /

# Créer un snapshot (lecture/écriture)
sudo btrfs subvolume snapshot / /snapshots/root-$(date +%Y%m%d)

# Créer un snapshot en LECTURE SEULE (recommandé pour les backups)
sudo btrfs subvolume snapshot -r / /snapshots/root-$(date +%Y%m%d)

# Lister les snapshots
sudo btrfs subvolume list -s /

# Supprimer un snapshot
sudo btrfs subvolume delete /snapshots/root-20260310

# Informations sur l'utilisation de l'espace
sudo btrfs filesystem usage /
sudo btrfs filesystem df /
```

#### Transfert de snapshots entre systèmes (send/receive)

```bash
# Envoyer un snapshot vers un autre disque ou serveur
sudo btrfs send /snapshots/root-20260310 | sudo btrfs receive /backup/btrfs/

# Envoi incrémental (uniquement les différences depuis un snapshot parent)
sudo btrfs send -p /snapshots/root-20260310 /snapshots/root-20260315 \
    | sudo btrfs receive /backup/btrfs/

# Envoi vers serveur distant via SSH
sudo btrfs send /snapshots/root-20260315 \
    | ssh user@backup-server "sudo btrfs receive /backup/btrfs/"

# Envoi compressé
sudo btrfs send /snapshots/root-20260315 \
    | zstd | ssh user@backup-server "zstd -d | sudo btrfs receive /backup/btrfs/"
```

### 6.3 Timeshift

#### Présentation

**Timeshift** est un outil graphique et en ligne de commande pour les snapshots système, comparable à la fonctionnalité *Time Machine* de macOS. Il supporte **Btrfs et RSYNC** comme backends.

| Backend | Mécanisme | Idéal pour |
|---------|-----------|-----------|
| **RSYNC** | Hardlinks rsync | Partitions ext4, xfs, tout FS standard |
| **Btrfs** | Snapshots Btrfs natifs | Partitions Btrfs uniquement |

#### Installation

```bash
# Debian/Ubuntu/Mint
sudo apt install timeshift

# RHEL/AlmaLinux (via COPR ou compilation)
sudo dnf install epel-release
sudo dnf copr enable hvr/timeshift
sudo dnf install timeshift

# Arch Linux (AUR)
yay -S timeshift
```

#### Utilisation en ligne de commande

```bash
# Créer un snapshot (backend rsync)
sudo timeshift --create --comments "Avant mise à jour système" --tags D

# Créer un snapshot Btrfs
sudo timeshift --btrfs --create --comments "Avant migration"

# Lister les snapshots disponibles
sudo timeshift --list

# Restauration d'un snapshot (interactif)
sudo timeshift --restore

# Restauration d'un snapshot spécifique (par nom)
sudo timeshift --restore --snapshot "2026-03-15_10-30-00"

# Supprimer un snapshot
sudo timeshift --delete --snapshot "2026-03-10_02-00-01"

# Configuration automatique (planification)
sudo timeshift --schedule --auto
```

#### Configuration de Timeshift (`/etc/timeshift/timeshift.json`)

```json
{
  "backup_device_uuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "parent_device_uuid": "",
  "do_first_run": "false",
  "btrfs_mode": "false",
  "include_btrfs_home_for_backup": "false",
  "stop_cron_emails": "true",
  "schedule_monthly": "false",
  "schedule_weekly": "false",
  "schedule_daily": "true",
  "schedule_hourly": "false",
  "schedule_boot": "false",
  "count_monthly": "2",
  "count_weekly": "3",
  "count_daily": "5",
  "count_hourly": "6",
  "count_boot": "5",
  "exclude": [
    "/root/**",
    "/home/**/.cache",
    "/home/**/.local/share/Trash"
  ]
}
```

### 6.4 Comparaison : LVM vs Btrfs vs Timeshift

| Critère | LVM Snapshots | Btrfs Snapshots | Timeshift (RSYNC) |
|---------|--------------|-----------------|-------------------|
| **FS requis** | Tout FS sur LVM | Btrfs uniquement | Tout FS |
| **Vitesse création** | Rapide | Instantané | Lent (copie) |
| **Espace initial** | Alloué à l'avance | Quasi nul (CoW) | Alloué (hardlinks) |
| **Cohérence données** | Crash-consistent | Crash-consistent | Pas toujours |
| **Granularité** | Volume logique | Sous-volume | Système complet |
| **Restauration partielle** | Via montage | Via montage | Via exploration |
| **Envoi réseau** | Non natif | Natif (send/recv) | Via rsync |
| **Interface** | CLI uniquement | CLI + outils tiers | CLI + GUI |
| **Recommandé pour** | Serveurs LVM | Systèmes Btrfs | Postes Debian/Ubuntu |

---

## 7. Tableau comparatif des commandes

### 7.1 Installation des outils selon la distribution

| Outil | Debian/Ubuntu/Mint | RHEL/CentOS/AlmaLinux | Arch Linux |
|-------|-------------------|----------------------|------------|
| **rsync** | `apt install rsync` | `dnf install rsync` | `pacman -S rsync` |
| **tar** | Préinstallé | Préinstallé | Préinstallé |
| **gzip** | Préinstallé | Préinstallé | Préinstallé |
| **bzip2** | `apt install bzip2` | Préinstallé | `pacman -S bzip2` |
| **xz** | `apt install xz-utils` | `dnf install xz` | Préinstallé |
| **zstd** | `apt install zstd` | `dnf install zstd` | `pacman -S zstd` |
| **restic** | `apt install restic` | `dnf install restic`¹ | `pacman -S restic` |
| **borgbackup** | `apt install borgbackup` | `dnf install borgbackup`¹ | `pacman -S borg` |
| **duplicati** | Via `.deb` officiel | Via `.rpm` officiel | `yay -S duplicati` |
| **bacula-client** | `apt install bacula-fd` | `dnf install bacula-client`¹ | `pacman -S bacula` |
| **bacula-server** | `apt install bacula` | `dnf install bacula-director bacula-storage`¹ | `yay -S bacula-server` |
| **timeshift** | `apt install timeshift` | Via COPR² | `yay -S timeshift` |
| **lvm2** | `apt install lvm2` | `dnf install lvm2` | `pacman -S lvm2` |
| **btrfs-progs** | `apt install btrfs-progs` | `dnf install btrfs-progs` | Préinstallé |
| **pv** | `apt install pv` | `dnf install pv`¹ | `pacman -S pv` |

> ¹ Nécessite le dépôt **EPEL** : `sudo dnf install epel-release`  
> ² COPR : `sudo dnf copr enable hvr/timeshift && sudo dnf install timeshift`

### 7.2 Commandes de gestion des paquets (rappel)

```bash
# ─── Debian / Ubuntu / Mint ───────────────────────────────────────
sudo apt update && sudo apt install <paquet>
sudo apt search <terme>
sudo apt show <paquet>

# ─── RHEL / CentOS / AlmaLinux ────────────────────────────────────
sudo dnf install epel-release          # Activer EPEL (prérequis pour de nombreux outils)
sudo dnf install <paquet>
sudo dnf search <terme>
sudo dnf info <paquet>

# ─── Arch Linux ───────────────────────────────────────────────────
sudo pacman -S <paquet>                # Dépôts officiels
yay -S <paquet>                        # AUR (Arch User Repository)
sudo pacman -Ss <terme>                # Recherche
sudo pacman -Si <paquet>               # Informations
```

### 7.3 Récapitulatif des opérations de backup par outil

| Opération | rsync | tar | restic | borg |
|-----------|-------|-----|--------|------|
| **Créer backup** | `rsync -av src/ dst/` | `tar -czf arch.tar.gz src/` | `restic backup src/` | `borg create repo::nom src/` |
| **Lister backups** | `ls -lh dst/` | `tar -tzf arch.tar.gz` | `restic snapshots` | `borg list repo` |
| **Restaurer** | `rsync -av dst/ src/` | `tar -xzf arch.tar.gz -C /` | `restic restore latest --target /` | `borg extract repo::nom` |
| **Vérifier intégrité** | `--checksum` | `tar -tzf arch.tar.gz` | `restic check` | `borg check repo` |
| **Pruning/Rétention** | Manuel | Manuel | `restic forget --prune` | `borg prune` |
| **Chiffrement** | Non natif | Non natif | ✓ Toujours actif | ✓ Optionnel |
| **Déduplication** | Partielle (hardlinks) | ✗ | ✓ | ✓ |
| **Compression** | `-z` (gzip) | `-z -j -J --zstd` | Automatique | lz4, zstd, lzma |
| **Backup réseau** | Via SSH | Via SSH/pipes | ✓ SFTP, S3... | ✓ SSH, S3... |

---

## Annexes

### A. Fichier `/etc/rsync-excludes.txt` de référence

```
# Caches et fichiers temporaires
*.cache/
*.tmp
*.swap
*.swp
*~

# Répertoires système à exclure des backups utilisateur
/proc/
/sys/
/dev/
/run/
/tmp/
/var/tmp/

# Données régénérables
node_modules/
__pycache__/
*.pyc
.git/objects/
target/          # Rust/Maven build

# Navigateurs
.mozilla/cache/
.config/google-chrome/Default/Cache/
.config/chromium/Default/Cache/

# Logs (souvent exclus des backups applicatifs)
*.log
*.log.*
```

### B. Checklist de validation d'une stratégie de backup

```
□ Stratégie 3-2-1 respectée (3 copies, 2 supports, 1 hors site)
□ Chiffrement activé pour les données sensibles
□ Sauvegardes planifiées et automatisées (cron/systemd timer)
□ Logs de sauvegarde centralisés et supervisés
□ Politique de rétention définie et appliquée
□ Procédure de restauration documentée
□ Test de restauration effectué et validé (fréquence : mensuelle recommandée)
□ Clés de chiffrement sauvegardées hors dépôt
□ Alertes en cas d'échec de sauvegarde configurées
□ Espace disque de destination supervisé
```

### C. Ressources officielles

| Outil | Documentation |
|-------|--------------|
| rsync | https://rsync.samba.org/documentation.html |
| tar | `man tar` / https://www.gnu.org/software/tar/manual/ |
| restic | https://restic.readthedocs.io |
| BorgBackup | https://borgbackup.readthedocs.io |
| Bacula | https://www.bacula.org/documentation/ |
| Timeshift | https://github.com/linuxmint/timeshift |
| Btrfs | https://btrfs.readthedocs.io |

---

*Document généré pour usage professionnel — SysAdmin Linux Reference Series*
