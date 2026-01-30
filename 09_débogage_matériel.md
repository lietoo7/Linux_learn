# Guide Complet : Maîtriser le Débogage Matériel via le Shell sous Linux

## Table des matières

1. Introduction et concepts fondamentaux
2. RAM - Mémoire vive
3. CPU - Processeur
4. Disques durs et stockage
5. Carte réseau
6. Écran et affichage
7. Clavier
8. USB et périphériques
9. Architecture complète du système
10. Méthodologie de diagnostic

---

## 1. Introduction et concepts fondamentaux

### 1.1 Comment Linux voit votre matériel

Linux utilise une philosophie simple : "Tout est fichier". Votre matériel est représenté par des fichiers dans des répertoires spéciaux :

- **`/dev/`** : Fichiers de périphériques (devices)
  - `/dev/sda` = premier disque SATA
  - `/dev/ttyUSB0` = premier port série USB
  - `/dev/input/` = périphériques d'entrée (clavier, souris)

- **`/sys/`** : Système de fichiers virtuel exposant les informations du kernel sur le matériel
  - `/sys/class/` : Classification par type de périphérique
  - `/sys/devices/` : Arborescence physique des périphériques

- **`/proc/`** : Informations sur les processus ET le système
  - `/proc/cpuinfo` : informations CPU
  - `/proc/meminfo` : informations mémoire

**Pourquoi c'est important** : Comprendre cette structure vous permet de savoir OÙ chercher l'information et comment le système organise le matériel.

### 1.2 Les bus de communication

Votre ordinateur utilise plusieurs "autoroutes" pour faire communiquer les composants :

- **PCI/PCIe (Peripheral Component Interconnect Express)** :
  - Bus haute vitesse pour cartes graphiques, WiFi, SSD NVMe
  - Commande : `lspci`

- **USB (Universal Serial Bus)** :
  - Bus pour périphériques externes et certains internes
  - Commande : `lsusb`

- **SATA (Serial ATA)** :
  - Bus pour disques durs et lecteurs optiques
  - Visible via `lsblk` ou `fdisk -l`

### 1.3 Les drivers (pilotes)

Un driver est un traducteur entre le kernel Linux et votre matériel. Sans driver :
- Le matériel existe physiquement
- Mais Linux ne sait pas lui parler

**Comment vérifier les drivers** :

```bash
# Lister les modules kernel chargés (= drivers actifs)
lsmod

# Exemple de sortie :
# Module                  Size  Used by
# iwlwifi               282624  1 iwlmvm    → Driver WiFi Intel
# e1000e                278528  0           → Driver Ethernet Intel
```

**Comprendre la sortie** :
- **Module** : nom du driver
- **Size** : taille en mémoire
- **Used by** : combien de fois utilisé et par quoi

---

## 2. RAM - Mémoire vive

### 2.1 Qu'est-ce que la RAM ?

La RAM (Random Access Memory) est la mémoire temporaire de votre ordinateur :
- **Volatile** : perd son contenu à l'extinction
- **Rapide** : accès en nanosecondes
- **Limitée** : 4-16GB typiquement sur un T420

### 2.2 Architecture mémoire

Votre T420 a 2 slots SO-DIMM (Small Outline Dual In-line Memory Module) :
- **Maximum** : 8GB (2x 4GB)
- **Type** : DDR3 PC3-10600 (1333MHz) ou PC3-12800 (1600MHz)
- **Format** : 204 pins

### 2.3 Diagnostic approfondi

#### Commande basique : free

```bash
free -h
```

**Sortie typique** :
```
              total        used        free      shared  buff/cache   available
Mem:           7.7G        2.3G        3.1G        234M        2.3G        4.8G
Swap:          4.0G          0B        4.0G
```

**Décryptage ligne par ligne** :
- **total** : RAM physique totale (7.7GB sur 8GB installés - une partie est réservée)
- **used** : mémoire activement utilisée par les applications (2.3GB)
- **free** : mémoire complètement inutilisée (3.1GB)
- **shared** : mémoire partagée entre processus (234MB)
- **buff/cache** : mémoire utilisée pour optimiser les performances
  - buffers : données en attente d'écriture sur disque
  - cache : fichiers récemment lus gardés en mémoire
  - **IMPORTANT** : Cette mémoire est libérable instantanément si nécessaire !
- **available** : mémoire réellement disponible pour nouvelles applications (4.8GB)
  - C'est le chiffre le plus important !
- **Swap** : espace disque utilisé comme mémoire "de secours"
  - Si swap utilisé = votre RAM est saturée = système ralenti

**L'option -h** : "human-readable" - affiche en GB/MB au lieu d'octets

#### Commande avancée : dmidecode

```bash
sudo dmidecode -t memory
```

**Pourquoi sudo ?** : dmidecode lit le BIOS/UEFI, zone protégée nécessitant les droits root.

**Sortie typique** :
```
Handle 0x0005, DMI type 16, 23 bytes
Physical Memory Array
        Location: System Board Or Motherboard
        Use: System Memory
        Error Correction Type: None
        Maximum Capacity: 8 GB
        Number Of Devices: 2

Handle 0x0006, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x0005
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 4096 MB
        Form Factor: SODIMM
        Locator: ChannelA-DIMM0
        Bank Locator: BANK 0
        Type: DDR3
        Speed: 1333 MT/s
        Manufacturer: Samsung
        Serial Number: 12345678
```

**Décryptage** :
- **Physical Memory Array** : Informations sur la carte mère
  - Maximum Capacity: 8 GB : maximum supporté
  - Number Of Devices: 2 : 2 slots disponibles
  - Error Correction Type: None : pas de ECC (correction d'erreurs)
- **Memory Device** : Informations sur CHAQUE barrette
  - Size: 4096 MB : taille de cette barrette (4GB)
  - Form Factor: SODIMM : format laptop (vs DIMM pour desktop)
  - Locator: ChannelA-DIMM0 : emplacement physique (slot 1)
  - Type: DDR3 : génération de RAM
  - Speed: 1333 MT/s : fréquence (MHz)
  - Manufacturer: Samsung : fabricant
  - Serial Number : numéro unique de la barrette

**Astuce pratique** : Si vous avez un slot vide, `Size: No Module Installed` apparaîtra.

#### Vérifier les erreurs mémoire

```bash
sudo journalctl -k | grep -i "memory\|edac"
```

**Explication** :
- `journalctl -k` : affiche les logs du kernel
- `grep -i` : filtre (case insensitive) pour "memory" OU "edac"
- **EDAC** : Error Detection And Correction - système de détection d'erreurs RAM

**Ce que vous cherchez** :
- Messages comme "EDAC MC0: UE" = Uncorrectable Error (barrette défectueuse !)
- "MCE" = Machine Check Exception (erreur matérielle grave)

### 2.4 Tests de RAM

#### Memtest86+ (le plus fiable)

Depuis le GRUB au démarrage - test HORS système d'exploitation :

```bash
# Installer
sudo dnf install memtest86+

# Au prochain reboot, choisir "Memory test" dans le menu GRUB
```

**Pourquoi hors OS ?** : Teste toute la RAM sans interférence du système.

**Durée** : Minimum 1 passe complète (2-3h pour 8GB). Idéalement : nuit complète.

**Interprétation** :
- 0 erreur : RAM saine
- 1+ erreur : Barrette défectueuse ou mal installée

#### memtester (test rapide sous OS)

```bash
# Installer
sudo dnf install memtester

# Tester 1GB pendant 5 passes
sudo memtester 1G 5
```

**Explication** :
- **1G** : quantité à tester (limitée car l'OS tourne)
- **5** : nombre de passes (répétitions du test)

**Patterns de test** :
- **Stuck Address** : vérifie que chaque cellule mémoire est accessible
- **Random Value** : écrit/lit des valeurs aléatoires
- **XOR comparison** : détecte les bits "collés"

### 2.5 Installation physique de RAM

#### Avant l'installation

**RÈGLE D'OR** : L'électricité statique TUE les composants électroniques !

**Préparation** :
1. Éteignez : `sudo shutdown -h now`
2. Débranchez l'alimentation
3. Retirez la batterie
4. Déchargez l'électricité statique :
   - Touchez une surface métallique reliée à la terre (radiateur, robinet)
   - Idéal : bracelet antistatique

#### Localisation sur T420

Le T420 a ses slots RAM sous un capot au dos :
- Dévissez la trappe marquée d'une icône RAM
- Les slots sont en "couloir" : un au-dessus de l'autre

#### Installation pas à pas

1. **Retrait (si remplacement)** :
   - Écartez les clips métalliques latéraux
   - La barrette se soulève à 45°
   - Tirez doucement dans l'axe

2. **Installation** :
   - Vérifiez l'encoche sur la barrette (déportée, pas au centre)
   - Insérez à 45° dans le slot
   - Appuyez jusqu'à entendre un "clic" (clips verrouillés)
   - La barrette doit être horizontale

3. **Vérification post-installation** :

```bash
# Après redémarrage
free -h
sudo dmidecode -t memory | grep -i size
```

**Compatibilité** :
- DDR3 uniquement (DDR4 ne rentre physiquement pas)
- Mixage possible mais déconseillé : préférez 2 barrettes identiques
- Fréquence : le système s'adaptera à la plus lente

---

## 3. CPU - Processeur

### 3.1 Architecture du processeur

Le CPU est le cerveau de l'ordinateur. Le T420 utilise typiquement :
- Intel Core i5-2520M ou i7-2620M (génération Sandy Bridge)
- **Architecture** : x86_64 (64 bits)
- **Cores** : 2 cœurs physiques
- **Threads** : 4 threads logiques (Hyper-Threading)

**Qu'est-ce qu'un thread ?** Imaginez un chef cuisinier (core) qui peut gérer 2 recettes en même temps (threads) en alternant rapidement les tâches.

### 3.2 Informations détaillées

#### Commande lscpu

```bash
lscpu
```

**Sortie typique** :
```
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
Model name:          Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
CPU MHz:             1245.678
CPU max MHz:         3200.0000
CPU min MHz:         800.0000
BogoMIPS:            4988.67
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            3072K
Flags:               fpu vme de pse ... (très longue liste)
```

**Décryptage ligne par ligne** :
- **Architecture: x86_64** :
  - Processeur 64 bits (peut exécuter logiciels 32 et 64 bits)
  - Comparé à ARM (smartphones) ou x86 (anciens 32 bits)
- **CPU(s): 4** :
  - Le système voit 4 CPU logiques
  - Calcul : 2 cores × 2 threads = 4
- **Thread(s) per core: 2** :
  - Hyper-Threading activé
  - Chaque cœur physique simule 2 cœurs logiques
- **CPU MHz: 1245.678** :
  - Fréquence ACTUELLE (change en temps réel !)
  - Varie entre min et max selon la charge
- **CPU max MHz: 3200** :
  - Fréquence Turbo Boost maximale
  - Atteinte sous forte charge (1 cœur)
- **CPU min MHz: 800** :
  - Fréquence minimale (économie d'énergie au repos)
  - Technologie : Intel SpeedStep
- **BogoMIPS: 4988.67** :
  - "Bogus MIPS" = mesure approximative de vitesse
  - Utile pour comparer des systèmes, pas une métrique absolue
- **Cache L1/L2/L3** :
  - **L1** : Cache le plus rapide mais le plus petit (32KB)
    - Divisé en L1d (données) et L1i (instructions)
  - **L2** : Cache intermédiaire (256KB par cœur)
  - **L3** : Cache partagé entre cœurs (3MB total)
  - **Analogie** : L1 = poche, L2 = sac à dos, L3 = coffre de voiture, RAM = garage
- **Flags** : Capacités du CPU
  - `sse, sse2, sse4_2` : Instructions vectorielles (calculs parallèles)
  - `aes` : Chiffrement AES matériel (plus rapide)
  - `vmx` : Virtualisation matérielle (VT-x)
  - `lm` : Long Mode = 64 bits supporté

#### Commande /proc/cpuinfo

```bash
cat /proc/cpuinfo
```

Informations similaires mais format différent, un bloc par CPU logique :

```
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 42
model name      : Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
stepping        : 7
microcode       : 0x2f
cpu MHz         : 1896.582
cache size      : 3072 KB
physical id     : 0
siblings        : 4
core id         : 0
cpu cores       : 2
apicid          : 0
fpu             : yes
bogomips        : 4988.67
clflush size    : 64
cache_alignment : 64
flags           : fpu vme de pse ...
```

**Nouveaux champs importants** :
- **processor: 0** : ID du CPU logique (0 à 3 sur votre système)
- **vendor_id: GenuineIntel** :
  - Fabricant (vs AuthenticAMD pour AMD)
- **cpu family: 6, model: 42** :
  - Identifiants internes Intel
  - Model 42 = Sandy Bridge (2ème génération Core)
- **stepping: 7** :
  - Révision du processeur (corrections de bugs)
- **microcode: 0x2f** :
  - Version du microcode (firmware CPU)
  - Updatable via : `sudo dnf install microcode_ctl`
- **physical id: 0** :
  - ID du socket physique
  - Toujours 0 sur laptop (1 seul CPU soudé)
- **core id: 0 ou 1** :
  - ID du cœur physique
  - Permet d'identifier les threads d'un même cœur

**Astuce : Compter les cœurs physiques** :

```bash
cat /proc/cpuinfo | grep "core id" | sort -u | wc -l
```

### 3.3 Surveillance de la température

#### Installation des sensors

```bash
sudo dnf install lm_sensors
sudo sensors-detect
```

**Le script sensors-detect** :
- Scanne tous les capteurs de température disponibles
- Propose de charger les modules nécessaires
- Répondez YES à tout sauf si vous savez ce que vous faites

**Pourquoi certaines questions ?** :
- Certains vieux chipsets ont des bugs de détection
- Le script vérifie la sécurité avant d'activer chaque capteur

#### Lecture des températures

```bash
sensors
```

**Sortie typique** :
```
coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +52.0°C  (high = +86.0°C, crit = +100.0°C)
Core 0:        +50.0°C  (high = +86.0°C, crit = +100.0°C)
Core 1:        +52.0°C  (high = +86.0°C, crit = +100.0°C)

thinkpad-isa-0000
Adapter: ISA adapter
fan1:        3200 RPM
temp1:        +52.0°C
temp2:        +48.0°C
temp3:         +0.0°C
```

**Interprétation** :
- **Package id 0: +52°C** :
  - Température globale du CPU
  - Zones :
    - < 60°C : Excellent (utilisation légère)
    - 60-80°C : Normal (charge moyenne)
    - 80-90°C : Attention (charge élevée, vérifier ventilation)
    - 90°C+ : DANGER (throttling imminent)
- **high = +86°C** :
  - Température où le CPU commence à réduire sa fréquence (throttling)
- **crit = +100°C** :
  - Température d'extinction d'urgence (protection matérielle)
- **Core 0/1** : Température de chaque cœur physique
- **fan1: 3200 RPM** :
  - Vitesse du ventilateur en tours/minute
  - Normal au repos : 2000-3000 RPM
  - Charge élevée : 4000-5000 RPM

**Surveillance en temps réel** :

```bash
watch -n 1 sensors
```

- Rafraîchit toutes les 1 seconde
- Utile pendant un stress test

### 3.4 Fréquence du CPU

#### Monitoring de la fréquence

```bash
watch -n 1 "grep 'MHz' /proc/cpuinfo"
```

**Pourquoi la fréquence varie ?** :
- Intel SpeedStep : économie d'énergie
- Au repos : 800 MHz (minimum)
- Sous charge : monte progressivement
- Turbo Boost : peut dépasser la fréquence de base (2.5 → 3.2 GHz)

#### Vérifier le gouverneur de fréquence :

```bash
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

**Gouverneurs disponibles** :
- **powersave** : maintient fréquence basse (économie maximale)
- **performance** : maintient fréquence maximale
- **ondemand** : ajuste selon la charge (défaut recommandé)
- **schedutil** : gouverneur moderne basé sur le scheduler

**Changer le gouverneur** :

```bash
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### 3.5 Tests de performance

#### Test de charge : stress

```bash
sudo dnf install stress

# Stresser 2 cœurs pendant 60 secondes
stress --cpu 2 --timeout 60s
```

**Options utiles** :
- `--cpu N` : nombre de workers CPU
- `--io N` : workers I/O (disque)
- `--vm N` : workers mémoire
- `--timeout Xs` : durée

**Pendant le test, surveillez** :

```bash
# Terminal 1
stress --cpu 4 --timeout 120s

# Terminal 2
watch -n 1 "sensors && grep MHz /proc/cpuinfo | head -4"
```

**Ce que vous devez observer** :
1. Fréquence monte vers le maximum
2. Température augmente
3. Ventilateur accélère
4. Après quelques secondes, équilibre thermique atteint

#### Benchmark : sysbench

```bash
sudo dnf install sysbench

# Test CPU : calcul de nombres premiers
sysbench cpu --threads=4 --time=60 run
```

**Sortie** :
```
CPU speed:
    events per second:   345.67

General statistics:
    total time:           60.0012s
    total number of events: 20741

Latency (ms):
         min:        11.23
         avg:        11.58
         max:        15.89
```

**Interprétation** :
- **events per second** : performance globale
  - Plus élevé = mieux
  - Comparez avant/après changement matériel
- **Latency avg** : temps moyen par calcul
  - Plus bas = mieux

### 3.6 Informations avancées

#### Vérifier la virtualisation

```bash
lscpu | grep Virtualization
```

**Résultat attendu** :
```
Virtualization:      VT-x
```

- **VT-x** (Intel) ou **AMD-V** (AMD) : support de la virtualisation matérielle
- Nécessaire pour VMware, VirtualBox, KVM avec bonnes performances
- Si absent : vérifiez dans le BIOS (peut être désactivé)

#### Vérifier les vulnérabilités

```bash
lscpu | grep Vulnerability
```

**Exemple** :
```
Vulnerability Meltdown:         Mitigation; PTI
Vulnerability Spectre v1:       Mitigation; usercopy/swapgs barriers
Vulnerability Spectre v2:       Mitigation; Full generic retpoline
```

- **Meltdown/Spectre** : vulnérabilités découvertes en 2018
- **Mitigation** : corrections appliquées (légère perte de performance)
- **PTI** : Page Table Isolation

---

## 4. Disques durs et stockage

### 4.1 Types de stockage

#### HDD (Hard Disk Drive)

**Fonctionnement mécanique** :
- Plateaux magnétiques tournant à 5400 ou 7200 RPM
- Têtes de lecture/écriture se déplaçant physiquement
- **Avantages** : Capacité élevée, prix bas
- **Inconvénients** : Lent, fragile aux chocs, bruyant

#### SSD (Solid State Drive)

**Fonctionnement électronique** :
- Mémoire flash (comme une clé USB géante)
- Aucune pièce mobile
- **Avantages** : Très rapide, silencieux, résistant aux chocs
- **Inconvénients** : Prix plus élevé, usure limitée (cycles d'écriture)

**Interfaces** :
- **SATA** : 6 Gb/s max (votre T420)
- **NVMe** : jusqu'à 32 Gb/s (nécessite slot M.2, absent sur T420)

### 4.2 Identification des disques

#### Commande lsblk

```bash
lsblk
```

**Sortie typique** :
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 232.9G  0 disk 
├─sda1   8:1    0   512M  0 part /boot
├─sda2   8:2    0  97.7G  0 part /
└─sda3   8:3    0   7.8G  0 part [SWAP]
sr0     11:0    1  1024M  0 rom
```

**Décryptage** :
- **NAME** : Nom du périphérique
  - `sda` : premier disque SATA (a = 1er, b = 2ème, etc.)
  - `sda1, sda2, sda3` : partitions sur sda
  - `sr0` : lecteur optique (DVD)
- **MAJ:MIN** : Major:Minor numbers
  - Identifiants uniques du kernel pour chaque périphérique
  - 8:0 = disque SCSI/SATA, 8:1 = première partition
- **RM** : Removable
  - 0 = non amovible
  - 1 = amovible (USB, carte SD)
- **SIZE** : Taille
  - 232.9G = capacité du disque
  - 512M, 97.7G = taille de chaque partition
- **RO** : Read-Only
  - 0 = lecture/écriture
  - 1 = lecture seule
- **TYPE** : Type
  - disk = disque entier
  - part = partition
  - rom = lecteur optique
- **MOUNTPOINT** : Point de montage
  - `/` = partition racine (système)
  - `/boot` = fichiers de démarrage (kernel, initramfs)
  - `[SWAP]` = espace d'échange (RAM virtuelle)

**Options utiles** :

```bash
lsblk -f  # Affiche les systèmes de fichiers
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT  # Colonnes personnalisées
```

#### Commande fdisk

```bash
sudo fdisk -l
```

**Sortie** :
```
Disk /dev/sda: 232.9 GiB, 250059350016 bytes, 488397168 sectors
Disk model: Samsung SSD 850
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x12345678

Device     Boot     Start       End   Sectors  Size Id Type
/dev/sda1  *         2048   1050623   1048576  512M 83 Linux
/dev/sda2         1050624 205824000 204773377 97.7G 83 Linux
/dev/sda3       205824001 222220287  16396287  7.8G 82 Linux swap
```

**Nouvelles informations** :
- **Disk model: Samsung SSD 850** :
  - Modèle exact du disque
  - Utile pour vérifier la compatibilité, chercher des firmware updates
- **Sectors: 488397168** :
  - Nombre total de secteurs
  - 1 secteur = 512 octets traditionnellement
- **Sector size (logical/physical): 512 bytes / 512 bytes** :
  - Taille logique : ce que l'OS voit
  - Taille physique : taille réelle sur le disque
  - Certains disques modernes : 512/4096 (Advanced Format)
- **Disklabel type: dos** :
  - Type de table de partition
  - dos = MBR (Master Boot Record) - ancien standard
  - gpt = GPT (GUID Partition Table) - standard moderne
  - MBR limite : 4 partitions primaires max, 2TB max
- **Boot : \*** :
  - Partition bootable (contient le GRUB)
- **Start/End** : Secteurs de début et fin
  - Détermine l'emplacement physique sur le disque
- **Id : Type** :
  - 83 = Linux filesystem
  - 82 = Linux swap
  - 7 = NTFS (Windows)
  - ef = EFI System

### 4.3 Santé du disque (SMART)

#### Qu'est-ce que SMART ?

**S.M.A.R.T.** = Self-Monitoring, Analysis and Reporting Technology

- Système intégré au disque
- Surveille les paramètres de santé
- Prévient les pannes imminentes

#### Vérification SMART

```bash
sudo dnf install smartmontools
sudo smartctl -a /dev/sda
```

**Sortie (extraits importants)** :
```
=== START OF INFORMATION SECTION ===
Model Family:     Samsung 850 EVO
Device Model:     Samsung SSD 850 EVO 250GB
Serial Number:    S21NNSAG123456L
Firmware Version: EMT02B6Q
User Capacity:    250,059,350,016 bytes [250 GB]
Rotation Rate:    Solid State Device
SATA Version is:  SATA 3.1, 6.0 Gb/s
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  5 Reallocated_Sector_Ct   0x0033   100   100   010    Pre-fail  Always       -       0
  9 Power_On_Hours          0x0032   099   099   000    Old_age   Always       -       1234
 12 Power_Cycle_Count       0x0032   099   099   000    Old_age   Always       -       567
177 Wear_Leveling_Count     0x0013   099   099   000    Pre-fail  Always       -       1
179 Used_Rsvd_Blk_Cnt_Tot   0x0013   100   100   010    Pre-fail  Always       -       0
181 Program_Fail_Cnt_Total  0x0032   100   100   010    Old_age   Always       -       0
182 Erase_Fail_Count_Total  0x0032   100   100   010    Old_age   Always       -       0
183 Runtime_Bad_Block       0x0013   100   100   010    Pre-fail  Always       -       0
187 Uncorrectable_Error_Cnt 0x0032   100   100   000    Old_age   Always       -       0
190 Airflow_Temperature_Cel 0x0032   074   051   000    Old_age   Always       -       26
195 ECC_Error_Rate          0x001a   200   200   000    Old_age   Always       -       0
199 CRC_Error_Count         0x003e   100   100   000    Old_age   Always       -       0
235 POR_Recovery_Count      0x0012   099   099   000    Old_age   Always       -       45
241 Total_LBAs_Written      0x0032   099   099   000    Old_age   Always       -       123456789
```

**Indicateurs CRITIQUES** :

1. **SMART overall-health: PASSED** :
   - PASSED = Disque sain
   - FAILED = Remplacez IMMÉDIATEMENT le disque !

2. **Reallocated_Sector_Ct (ID 5)** :
   - Secteurs défectueux remplacés
   - VALUE: 100 = aucun secteur réalloué (parfait)
   - Seuil critique : > 0 = début de dégradation
   - > 10 = disque en fin de vie

3. **Power_On_Hours (ID 9)** :
   - Heures d'utilisation : 1234h
   - Espérance de vie SSD : 20,000 - 50,000h
   - HDD : 20,000 - 40,000h

4. **Wear_Leveling_Count (ID 177)** :
   - Usure globale du SSD
   - 99% restant (excellent)
   - < 10% = envisager le remplacement

5. **Used_Rsvd_Blk_Cnt_Tot (ID 179)** :
   - Blocs de réserve utilisés
   - 0 = aucun (parfait)
   - Augmentation = usure avancée

6. **Uncorrectable_Error_Cnt (ID 187)** :
   - Erreurs non corrigées
   - DOIT être 0
   - > 0 = perte de données possible !

7. **Airflow_Temperature_Cel (ID 190)** :
   - Température : 26°C
   - Normal : 20-50°C
   - > 60°C = ventilation insuffisante

8. **Total_LBAs_Written (ID 241)** :
   - Quantité de données écrites (en blocs)
   - Divisez par 2048 pour obtenir des GB
   - Calcul TBW (TeraBytes Written) : santé du SSD

**Décryptage des colonnes** :
- **VALUE** : Valeur actuelle normalisée (0-100 ou 0-253)
- **WORST** : Pire valeur jamais atteinte
- **THRESH** : Seuil critique (si VALUE < THRESH = FAILED)
- **TYPE** :
  - Pre-fail : Prédictif de panne imminente
  - Old_age : Usure normale
- **WHEN_FAILED** : Quand l'attribut a échoué (idéalement "-")
- **RAW_VALUE** : Valeur brute du fabricant

#### Tests SMART

```bash
# Test court (2 minutes)
sudo smartctl -t short /dev/sda

# Attendre 2 minutes puis voir résultat
sudo smartctl -l selftest /dev/sda

# Test long (30-120 minutes selon taille)
sudo smartctl -t long /dev/sda
```

**Résultat** :
```
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short offline       Completed without error       00%      1235         -
```

- **Completed without error** : Parfait !
- **Completed: read failure** : Secteurs défectueux
- **Aborted by host** : Test interrompu (normal si vous avez redémarré)

### 4.4 Performance du disque

#### Test de vitesse : hdparm

```bash
sudo dnf install hdparm
sudo hdparm -Tt /dev/sda
```

**Sortie** :
```
/dev/sda:
 Timing cached reads:   12000 MB in  2.00 seconds = 5998.23 MB/sec
 Timing buffered disk reads: 1284 MB in  3.00 seconds = 427.65 MB/sec
```

**Interprétation** :
- **Cached reads: 5998 MB/sec** :
  - Lecture depuis le cache mémoire
  - Teste la vitesse de la RAM et du bus, pas du disque
  - Normal : 3000-10000 MB/s
- **Buffered disk reads: 427 MB/sec** :
  - Lecture RÉELLE depuis le disque
  - HDD 7200 RPM : 80-160 MB/s
  - HDD 5400 RPM : 50-100 MB/s
  - SSD SATA : 200-550 MB/s (votre cas)
  - SSD NVMe : 1500-7000 MB/s

**Test d'écriture (ATTENTION : destructif sur partition vide !)** :

```bash
# NE PAS FAIRE sur partition système !
# Exemple sur une partition de test
sudo dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct
```

### 4.5 Gestion des partitions

#### Voir les UUID

```bash
sudo blkid
```

**Sortie** :
```
/dev/sda1: UUID="1234-5678" TYPE="vfat" PARTUUID="12345678-01"
/dev/sda2: UUID="abcd-efgh-ijkl-mnop" TYPE="ext4" PARTUUID="12345678-02"
/dev/sda3: UUID="wxyz-1234-5678-abcd" TYPE="swap" PARTUUID="12345678-03"
```

**UUID (Universally Unique IDentifier)** :
- Identifiant unique de la partition (ne change jamais)
- Utilisé dans `/etc/fstab` pour le montage automatique
- **Pourquoi UUID plutôt que /dev/sda1 ?** :
  - Si vous ajoutez un disque, sda peut devenir sdb
  - UUID reste constant

**TYPE : Système de fichiers** :
- **ext4** : Standard Linux (journalisé, robuste)
- **xfs** : Hautes performances (par défaut sur RHEL/AlmaLinux)
- **btrfs** : Avancé (snapshots, compression)
- **vfat** : FAT32 (compatible Windows/Mac/Linux)
- **ntfs** : Windows (lecture/écriture avec driver supplémentaire)

#### Espace disque

```bash
df -h
```

**Sortie** :
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        96G   45G   47G  49% /
/dev/sda1       511M  128M  384M  25% /boot
tmpfs           3.9G  2.3M  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /dev/shm
```

**Colonnes** :
- **Filesystem** : Partition
- **Size** : Taille totale
- **Used** : Espace utilisé
- **Avail** : Espace disponible (pas exactement Size - Used à cause des blocs réservés)
- **Use%** : Pourcentage d'utilisation
  - > 90% : Attention, espace faible
  - 100% : Plus d'espace libre, système instable !
- **Mounted on** : Point de montage

**Systèmes de fichiers virtuels** :
- **tmpfs** : Stockage en RAM (ultra-rapide, perdu au reboot)
  - `/run` : Fichiers temporaires du système
  - `/dev/shm` : Mémoire partagée

**Trouver les gros fichiers** :

```bash
# Taille de chaque répertoire dans /home
du -sh /home/*

# Top 10 des plus gros fichiers
sudo find / -type f -size +100M -exec du -h {} \; | sort -rh | head -10
```

### 4.6 Installation physique

#### Disque principal (slot SATA)

**Localisation T420** :
- Sous le laptop, trappe à vis (symbole disque)
- Ou : retirer le clavier, dévisser caddy

**Procédure** :
1. Éteignez et déchargez l'électricité statique
2. Retirez la batterie
3. Dévissez la trappe
4. Dégagez le caddy (rail métallique)
5. Dévissez le disque du caddy (4 vis latérales)
6. Installez le nouveau disque (mêmes vis)
7. Réinsérez le caddy jusqu'au "clic"

**Compatibilité** :
- **Format** : 2.5" (laptop standard)
- **Épaisseur** : 7mm ou 9.5mm (les deux fonctionnent)
- **Interface** : SATA III (6 Gb/s) rétrocompatible SATA II

#### Disque secondaire (Ultrabay)

**Utilisation créative** : Le T420 a un lecteur DVD amovible (Ultrabay). Vous pouvez le remplacer par un caddy HDD/SSD !

**Produit** : Cherchez "Ultrabay HDD Caddy T420" (12mm d'épaisseur)

**Installation** :
1. Dévissez la vis de verrouillage Ultrabay (logo DVD)
2. Glissez le lecteur DVD hors du slot
3. Retirez la façade du lecteur DVD
4. Installez la façade sur le caddy HDD
5. Installez le SSD/HDD dans le caddy
6. Insérez le caddy dans l'Ultrabay

**Configuration DUAL BOOT avancée** :
- SSD principal : AlmaLinux
- HDD secondaire : Windows ou stockage de données

---

## 5. Carte réseau

### 5.1 Architecture réseau

Votre T420 possède généralement :
1. **Carte Ethernet** : Intel 82579LM (1 Gigabit)
2. **Carte WiFi** : Intel Centrino Advanced-N 6205 (dual-band)

### 5.2 Identification

#### Lister les interfaces

```bash
ip link show
```

**Sortie typique** :
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:21:cc:d4:5e:7f brd ff:ff:ff:ff:ff:ff
3: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 84:3a:4b:12:34:56 brd ff:ff:ff:ff:ff:ff
```

**Décryptage** :
- **lo** : Loopback
  - Interface virtuelle (127.0.0.1)
  - Communication interne à la machine
  - Toujours présente
- **enp0s25** : Ethernet
  - Nouveau nommage : e(thernet) n(nom) p0(bus PCI) s25(slot)
  - Ancien nom : eth0
  - Pourquoi ce changement ? : Noms prévisibles et stables
- **wlp3s0** : WiFi
  - w(ireless) l(an) p3(bus PCI) s0(slot)
  - Ancien nom : wlan0
- **État** :
  - `<UP>` : Interface activée
  - `<LOWER_UP>` : Câble branché (Ethernet) ou connecté (WiFi)
  - `state UP` : Pleinement fonctionnelle
  - `state DOWN` : Désactivée
- **MTU 1500** : Maximum Transmission Unit
  - Taille maximale d'un paquet : 1500 octets
  - Standard Ethernet
  - Jumbo frames : jusqu'à 9000 (rare, réseaux spécialisés)
- **qdisc fq_codel** : Queue Discipline
  - Algorithme de gestion de la file d'attente
  - fq_codel : Fair Queuing avec CoDel (réduit latence)
- **link/ether 00:21:cc:d4:5e:7f** : Adresse MAC
  - Media Access Control
  - Adresse physique unique au monde
  - 6 octets en hexadécimal
  - 00:21:cc = OUI (Organizationally Unique Identifier) - identifie Intel

#### Informations IP

```bash
ip addr show
```

**Sortie** :
```
2: enp0s25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:21:cc:d4:5e:7f brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.42/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s25
       valid_lft 86234sec preferred_lft 86234sec
    inet6 fe80::221:ccff:fed4:5e7f/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

**Nouvelles infos** :
- **inet 192.168.1.42/24** :
  - Adresse IPv4 : 192.168.1.42
  - / 24 : Masque de sous-réseau (255.255.255.0)
    - 24 premiers bits = réseau
    - 8 derniers bits = hôtes (254 adresses disponibles)
- **brd 192.168.1.255** : Adresse de broadcast
  - Envoyer à tous les appareils du réseau
- **scope global** : Portée
  - global : routée vers Internet
  - link : locale au lien physique
  - host : locale à la machine
- **dynamic** : Adresse obtenue via DHCP
  - Vs static : configurée manuellement
- **valid_lft 86234sec** : Durée de validité
  - Bail DHCP : 86234 secondes (≈ 24h)
  - Renouvellement automatique
- **inet6 fe80::...** : Adresse IPv6 link-local
  - Autoconfiguration automatique
  - Communication locale seulement

#### Détails matériels

```bash
sudo lshw -C network
```

**Sortie** :
```
  *-network
       description: Ethernet interface
       product: 82579LM Gigabit Network Connection
       vendor: Intel Corporation
       physical id: 19
       bus info: pci@0000:00:19.0
       logical name: enp0s25
       version: 04
       serial: 00:21:cc:d4:5e:7f
       size: 1Gbit/s
       capacity: 1Gbit/s
       width: 32 bits
       clock: 33MHz
       capabilities: ethernet physical tp 10bt 10bt-fd 100bt 100bt-fd 1000bt-fd autonegotiation
       configuration: autonegotiation=on broadcast=yes driver=e1000e driverversion=5.14.0 duplex=full firmware=0.13-4 latency=0 link=yes multicast=yes port=twisted pair speed=1Gbit/s
       resources: irq:31 memory:f2500000-f251ffff memory:f253b000-f253bfff ioport:5080(size=32)
```

**Informations clés** :
- **product: 82579LM** : Modèle exact de la carte
- **vendor: Intel Corporation** : Fabricant
- **bus info: pci@0000:00:19.0** : Emplacement PCI
- **size: 1Gbit/s** : Vitesse actuelle de connexion
- **capacity: 1Gbit/s** : Vitesse maximale supportée
- **capabilities** :
  - 10bt, 100bt, 1000bt-fd : Supporte 10/100/1000 Mbps full-duplex
  - autonegotiation : Détection automatique de vitesse
  - tp : Twisted Pair (câble RJ45 standard)
- **driver=e1000e** : Driver utilisé
- **driverversion=5.14.0** : Version du kernel où le driver est compilé
- **duplex=full** : Communication bidirectionnelle simultanée
  - half : un sens à la fois (ancien)
- **link=yes** : Câble connecté

### 5.3 Tests de connectivité

#### Ping basique

```bash
ping -c 4 8.8.8.8
```

**Sortie** :
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=12.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=11.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=13.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=115 time=12.0 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 11.827/12.325/13.098/0.501 ms
```

**Analyse** :
- **8.8.8.8** : DNS Google (test de connexion Internet)
- **64 bytes** : Taille du paquet ICMP
- **icmp_seq** : Numéro de séquence (détecte paquets manquants)
- **ttl=115** : Time To Live
  - Nombre de sauts réseau restants (max 255)
  - 115 = environ 10-12 sauts depuis vous
- **time=12.4 ms** : Latence (ping)
  - < 20 ms : Excellent (réseau local ou proche)
  - 20-50 ms : Bon
  - 50-100 ms : Moyen
  - > 100 ms : Lent
  - > 300 ms : Très lent (réseau saturé ou distant)
- **0% packet loss** : Aucune perte de paquets (parfait)
  - > 5% : Problème réseau
- **rtt** : Round-Trip Time
  - min : meilleur ping
  - avg : moyenne (indicateur principal)
  - max : pire ping
  - mdev : écart-type (stabilité)

#### Traceroute

```bash
traceroute 8.8.8.8
```

**Sortie** :
```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  2.451 ms  2.123 ms  2.045 ms
 2  10.0.0.1 (10.0.0.1)  8.234 ms  8.123 ms  8.456 ms
 3  72.14.204.1 (72.14.204.1)  12.345 ms  12.123 ms  12.567 ms
 4  * * *
 5  8.8.8.8 (8.8.8.8)  13.456 ms  13.234 ms  13.678 ms
```

**Interprétation** :
- Chaque ligne = un saut (routeur traversé)
- 1 _gateway (192.168.1.1) : Votre box/routeur
- 2 10.0.0.1 : Premier routeur de votre FAI
- 3 72.14.204.1 : Routeur Google
- **4 \* \* \*** : Routeur ne répond pas (normal, sécurité)
- 5 8.8.8.8 : Destination atteinte

**3 temps affichés** : 3 paquets envoyés pour fiabilité

### 5.4 Vitesse et performances

#### ethtool

```bash
sudo ethtool enp0s25
```

**Sortie** :
```
Settings for enp0s25:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Supported pause frame use: No
        Supports auto-negotiation: Yes
        Supported FEC modes: Not reported
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes
        Speed: 1000Mb/s
        Duplex: Full
        Auto-negotiation: on
        Port: Twisted Pair
        Link detected: yes
```

**Points importants** :
- **Speed: 1000Mb/s** : Connecté à 1 Gigabit
  - Si 100Mb/s : Câble défectueux ou switch ancien
  - Si 10Mb/s : Gros problème !
- **Duplex: Full** : Communication bidirectionnelle
  - Half = problème de négociation
- **Link detected: yes** : Câble physiquement connecté

#### Statistiques d'interface

```bash
ip -s link show enp0s25
```

**Sortie** :
```
2: enp0s25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:21:cc:d4:5e:7f brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    12345678   9876     0       0       0       123     
    TX: bytes  packets  errors  dropped carrier collsns 
    98765432   8765     0       0       0       0
```

**RX (Receive - Réception)** :
- **bytes** : Données reçues (12.3 MB)
- **packets** : Nombre de paquets (9876)
- **errors** : Erreurs de transmission
  - > 0 : Problème physique (câble, carte)
- **dropped** : Paquets abandonnés (buffer plein)
- **overrun** : Carte surchargée (impossible de traiter)
- **mcast** : Paquets multicast

**TX (Transmit - Émission)** :
- **carrier** : Perte de signal physique
- **collsns** : Collisions (ancien Ethernet half-duplex)

**Si errors > 0** :
1. Vérifiez le câble
2. Testez un autre port sur le switch
3. Suspectez la carte réseau

### 5.5 WiFi spécifique

#### Scanner les réseaux

```bash
sudo nmcli dev wifi list
```

**Sortie** :
```
IN-USE  BSSID              SSID            MODE   CHAN  RATE        SIGNAL  BARS  SECURITY
*       AA:BB:CC:DD:EE:FF  MonReseau       Infra  6     130 Mbit/s  87      ▂▄▆█  WPA2
        11:22:33:44:55:66  Voisin-WiFi     Infra  11    54 Mbit/s   42      ▂▄__  WPA2
        77:88:99:AA:BB:CC  FreeWiFi        Infra  1     54 Mbit/s   35      ▂___  --
```

**Colonnes** :
- **IN-USE : \*** : Réseau actuellement connecté
- **BSSID** : Adresse MAC du point d'accès (AP)
- **SSID** : Nom du réseau
- **MODE: Infra** : Mode infrastructure (vs Ad-hoc)
- **CHAN** : Canal WiFi
  - 2.4 GHz : Canaux 1-13
    - Canaux sans chevauchement : 1, 6, 11
    - Évitez les autres pour réduire interférences
  - 5 GHz : Canaux 36-165
- **RATE** : Vitesse maximale négociée
- **SIGNAL** : Puissance du signal (0-100)
  - > 70 : Excellent
  - 50-70 : Bon
  - 30-50 : Moyen
  - < 30 : Faible (déconnexions possibles)
- **SECURITY** :
  - WPA2 : Sécurisé (AES)
  - WPA : Ancien, moins sécurisé
  - WEP : Obsolète, facilement cassable
  - -- : Ouvert (non sécurisé)

#### Informations de connexion WiFi

```bash
nmcli dev show wlp3s0
```

**Sortie (extraits)** :
```
GENERAL.DEVICE:                         wlp3s0
GENERAL.TYPE:                           wifi
GENERAL.HWADDR:                         84:3A:4B:12:34:56
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     MonReseau
IP4.ADDRESS[1]:                         192.168.1.50/24
IP4.GATEWAY:                            192.168.1.1
IP4.DNS[1]:                             192.168.1.1
WIFI-PROPERTIES.WPA-FLAGS:              0x188
WIFI-PROPERTIES.RSN-FLAGS:              0x188
WIFI-PROPERTIES.FREQ:                   2437 MHz
WIFI-PROPERTIES.SSID:                   MonReseau
WIFI-PROPERTIES.MODE:                   infrastructure
WIFI-PROPERTIES.RATE:                   130000 Kbit/s
WIFI-PROPERTIES.SIGNAL:                 87
```

- **FREQ: 2437 MHz** :
  - Fréquence WiFi
  - 2400-2483 MHz : bande 2.4 GHz (canal 1-13)
  - 5170-5825 MHz : bande 5 GHz
  - Calcul du canal : (2437 - 2407) / 5 = Canal 6
- **RATE: 130000 Kbit/s** : 130 Mbps (vitesse de liaison)

#### Qualité du signal en temps réel

```bash
watch -n 1 "cat /proc/net/wireless"
```

**Sortie** :
```
Inter-| sta-|   Quality        |   Discarded packets               | Missed | WE
 face | tus | link level noise |  nwid  crypt   frag  retry   misc | beacon | 22
wlp3s0: 0000   67.  -43.  -256        0      0      0      0      0        0
```

- **link: 67** : Qualité sur 70 (excellent)
- **level: -43** : Puissance en dBm
  - -30 dBm : Signal parfait (très proche)
  - -50 dBm : Excellent
  - -60 dBm : Bon
  - -70 dBm : Faible
  - -80 dBm : Très faible
  - < -90 dBm : Inutilisable
- **retry** : Paquets retransmis
  - > 0 : Interférences ou signal faible

### 5.6 Installation physique

#### Carte WiFi (T420)

**Localisation** :
- Sous le clavier (retirer 7 vis marquées)
- Carte Mini PCIe demi-hauteur

**Whitelist BIOS** : ⚠️ ATTENTION : Lenovo a une "whitelist" de cartes WiFi autorisées dans le BIOS !

- Seules certaines cartes Intel/Realtek sont acceptées
- Une carte non listée = refus de boot

**Solutions** :
1. Acheter une carte de la whitelist (Intel 6300, 6205, 7260)
2. Flasher un BIOS modifié (avancé, risqué)

**Installation** :
1. Retirez le clavier (7 vis)
2. Soulevez le clavier délicatement (nappes attachées)
3. Repérez la carte WiFi (coin supérieur droit)
4. Débranchez les 2 antennes (connecteurs minuscules)
   - Main : Câble noir
   - Aux : Câble blanc
5. Dévissez la vis de maintien
6. Retirez la carte à 30°
7. Insérez la nouvelle carte (encoches alignées)
8. Reconnectez les antennes (pas de force !)

---

## 6. Écran et affichage

### 6.1 Architecture graphique

Votre T420 a :
- **GPU intégré** : Intel HD Graphics 3000 (dans le CPU)
- **Écran** : 14.0" LED, 1366×768 ou 1600×900 (selon modèle)
- **Connecteurs** : VGA, DisplayPort

### 6.2 Informations sur la carte graphique

```bash
lspci | grep -i vga
```

**Sortie** :
```
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
```

- **00:02.0** : Adresse PCI
- **2nd Generation Core** : Sandy Bridge (2011)

**Détails complets** :

```bash
sudo lshw -C display
```

**Sortie** :
```
  *-display
       description: VGA compatible controller
       product: 2nd Generation Core Processor Family Integrated Graphics Controller
       vendor: Intel Corporation
       physical id: 2
       bus info: pci@0000:00:02.0
       version: 09
       width: 64 bits
       clock: 33MHz
       capabilities: vga_controller bus_master cap_list rom
       configuration: driver=i915 latency=0
       resources: irq:30 memory:f0000000-f03fffff memory:e0000000-efffffff ioport:5000(size=64)
```

- **driver=i915** : Driver Intel standard
- **memory** : Mémoire vidéo partagée avec la RAM système

### 6.3 Résolution et configuration

```bash
xrandr
```

**Sortie** :
```
Screen 0: minimum 8 x 8, current 1366 x 768, maximum 16384 x 16384
LVDS1 connected primary 1366x768+0+0 (normal left inverted right x axis y axis) 309mm x 174mm
   1366x768      60.00*+
   1024x768      60.00  
   800x600       60.32    56.25  
   640x480       59.94  
VGA1 disconnected (normal left inverted right x axis y axis)
DP1 disconnected (normal left inverted right x axis y axis)
```

**Décryptage** :
- **Screen 0** : Écran virtuel (peut contenir plusieurs sorties)
- **LVDS1** : Écran interne du laptop
  - LVDS : Low-Voltage Differential Signaling (interface LCD)
  - connected primary : Écran principal actif
  - 1366x768+0+0 : Résolution et position (x+y)
  - 309mm x 174mm : Taille physique (calculée via EDID)
- **1366x768 60.00\*+** :
  - Résolution disponible
  - 60.00 : Taux de rafraîchissement (Hz)
  - **\*** : Résolution actuelle
  - **+** : Résolution préférée (native)
- **VGA1, DP1** : Sorties vidéo externes (non connectées)

**Changer la résolution** :

```bash
xrandr --output LVDS1 --mode 1024x768
```

**Multi-écran** :

```bash
# Écran externe à droite de l'interne
xrandr --output VGA1 --auto --right-of LVDS1
```

### 6.4 Luminosité

#### Chemin système

```bash
ls /sys/class/backlight/
```

**Sortie** :
```
intel_backlight
```

**Lire la luminosité** :

```bash
cat /sys/class/backlight/intel_backlight/brightness
cat /sys/class/backlight/intel_backlight/max_brightness
```

**Exemple** :
```
5000    (actuelle)
7812    (maximum)
```

**Calcul du pourcentage** : 5000 / 7812 = 64%

**Modifier la luminosité** :

```bash
# 50% de 7812 = 3906
echo 3906 | sudo tee /sys/class/backlight/intel_backlight/brightness
```

**Astuce : Créer un alias dans ~/.bashrc** :

```bash
alias bright='echo $((7812 * $1 / 100)) | sudo tee /sys/class/backlight/intel_backlight/brightness'
```

**Utilisation** : `bright 75` (met à 75%)

#### Touches de fonction

Les touches Fn+F8/F9 ajustent la luminosité :
- Gérées par le daemon acpid
- Configuration dans `/etc/acpi/events/`

### 6.5 EDID (Extended Display Identification Data)

L'EDID contient les caractéristiques de votre écran :

```bash
sudo dnf install read-edid
sudo get-edid | parse-edid
```

**Sortie** :
```
EDID version: 1.3
Manufacturer: LEN (Lenovo)
Model: 40b0
Serial number: 0
Made in: week 0, 2011
Digital display
Maximum image size: 31 cm x 17 cm
Gamma: 2.20
Supported features: DPMS, active off, RGB color
Preferred mode: 1366x768 @ 60Hz
Native resolution: 1366x768
```

**Informations importantes** :
- **Manufacturer** : Fabricant de la dalle
- **Made in** : Date de fabrication
- **Gamma: 2.20** : Courbe de correction colorimérique
- **Native resolution** : Résolution physique de la dalle

### 6.6 Remplacement de l'écran

**Modèles compatibles T420** :
- **1366×768** : Standard (dalle TN, angles de vue limités)
- **1600×900** : Premium (dalle IPS possible, meilleurs angles)

**Upgrade possible** :
- Passer de 1366×768 à 1600×900
- Nécessite dalle compatible + inverter (si CCFL) ou pas (si LED)

**Procédure** :
1. Retirez la batterie
2. Dévissez les caches des charnières (avant du laptop)
3. Retirez les caches caoutchouc devant l'écran (6 points)
4. Dévissez les 6 vis cachées
5. Séparez le cadre de l'écran (clips plastiques)
6. Dévissez les supports métalliques de la dalle (4 vis)
7. Basculez la dalle, débranchez le câble LVDS
8. Installation inverse avec nouvelle dalle

**ATTENTION** :
- Dalle fragile (ne pas appuyer sur la surface)
- Câble LVDS délicat (pas de pliure)
- Vérifiez la compatibilité (connecteur 40 pins)

---

## 7. Clavier

### 7.1 Architecture du clavier

Le clavier est un périphérique d'entrée HID (Human Interface Device) :
- Connecté en interne via PS/2 (émulé sur USB)
- Gère les touches ET le TrackPoint

### 7.2 Identification

```bash
cat /proc/bus/input/devices | grep -A 5 keyboard
```

**Sortie** :
```
N: Name="AT Translated Set 2 keyboard"
P: Phys=isa0060/serio0/input0
S: Sysfs=/devices/platform/i8042/serio0/input/input3
U: Uniq=
H: Handlers=sysrq kbd event3 
B: PROP=0
```

**Explication** :
- **Name: AT Translated Set 2** :
  - Standard PC clavier
  - "AT" = IBM PC/AT (ancien standard toujours utilisé)
- **Phys: isa0060/serio0** :
  - Port PS/2 (adresse I/O 0x60)
  - serio0 : premier port série (clavier)
  - serio1 : second port (TrackPoint/touchpad)
- **Handlers: sysrq kbd event3** :
  - sysrq : Magic SysRq (touches d'urgence kernel)
  - kbd : Interface clavier standard
  - event3 : Fichier événement (/dev/input/event3)

### 7.3 Test des touches

#### evtest (outil interactif)

```bash
sudo dnf install evtest
sudo evtest
```

**Déroulement** :
1. Liste les périphériques disponibles
2. Choisissez le clavier (ex: /dev/input/event3)
3. Tapez des touches, voyez les événements

**Sortie exemple** :
```
Event: time 1234567890.123456, type 4 (EV_MSC), code 4 (MSC_SCAN), value 1c
Event: time 1234567890.123456, type 1 (EV_KEY), code 28 (KEY_ENTER), value 1
Event: time 1234567890.123456, -------------- SYN_REPORT ------------
```

**Décryptage** :
- **type 1 (EV_KEY)** : Événement touche
- **code 28 (KEY_ENTER)** : Touche Entrée
- **value 1** : Appuyée (0 = relâchée, 2 = maintenue)

**Utilité** :
- Tester des touches douteuses
- Vérifier les touches multimédia
- Diagnostiquer un clavier défectueux

#### xev (environnement graphique)

```bash
xev
```

- Ouvre une fenêtre
- Tapez dedans, voyez les événements X11
- Plus d'infos : keycodes X, modificateurs (Shift, Ctrl)

### 7.4 Disposition du clavier

```bash
# Voir la disposition actuelle
localectl status
```

**Sortie** :
```
   System Locale: LANG=fr_FR.UTF-8
       VC Keymap: fr
      X11 Layout: fr
     X11 Variant: oss
```

- **VC Keymap: fr** : Console virtuelle (TTY) en français
- **X11 Layout: fr** : Environnement graphique en français
- **X11 Variant: oss** : Variante (oss = avec touches mortes)

**Changer la disposition** :

```bash
sudo localectl set-keymap fr
sudo localectl set-x11-keymap fr
```

### 7.5 Rétro-éclairage du clavier

Le T420 n'a PAS de rétro-éclairage clavier de série.

**Option** : Clavier de T420s (version slim) compatible mais rare.

### 7.6 Remplacement du clavier

**Procédure T420** :
1. Éteignez et retirez la batterie
2. Poussez les 2 languettes au-dessus du clavier vers l'écran
3. Basculez le clavier vers vous (charnière en bas)
4. Débranchez le câble ruban (soulevez le loquet marron)
5. Installez le nouveau clavier (insérez le câble, verrouillez)
6. Replacez le clavier (clips en haut, languettes en bas)

**Compatibilité** :
- Cherchez "FRU 45N2211" (clavier FR)
- Ou équivalent US/UK selon votre besoin
- Compatible T420, T420i, T420s (vérifiez nappes)

---

## 8. USB et périphériques

### 8.1 Architecture USB

**Versions USB** :
- **USB 1.1** : 12 Mbit/s (obsolète)
- **USB 2.0** : 480 Mbit/s (60 MB/s)
- **USB 3.0** : 5 Gbit/s (625 MB/s) - connecteur bleu
- **USB 3.1/3.2** : 10-20 Gbit/s (absent sur T420)

**T420 dispose** :
- 2× USB 2.0 (côté gauche)
- 1× USB 2.0 + eSATA combo (côté gauche)
- 1× USB 3.0 (côté droit, ExpressCard avec adaptateur optionnel)

### 8.2 Lister les périphériques USB

```bash
lsusb
```

**Sortie** :
```
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 003: ID 04f2:b217 Chicony Electronics Co., Ltd Lenovo Integrated Camera
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 002: ID 0a5c:217f Broadcom Corp. BCM2045B (BDC-2.1)
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```

**Format** : `Bus XXX Device YYY: ID VVVV:PPPP Manufacturer Product`

- **Bus 001** : Contrôleur USB #1
- **Device 002** : Périphérique #2 sur ce bus
- **ID 8087:0024** :
  - 8087 : Vendor ID (Intel)
  - 0024 : Product ID (Hub USB)

**Périphériques internes** :
- **Integrated Rate Matching Hub** : Hub interne (multiplie les ports)
- **Lenovo Integrated Camera** : Webcam
- **Broadcom BCM2045B** : Bluetooth
- **root hub** : Contrôleur USB racine

**Version détaillée** :

```bash
lsusb -v
```

⚠️ Très verbeux (plusieurs pages par périphérique)

**Filtrer** :

```bash
lsusb -v -d 04f2:b217  # Détails de la webcam
```

### 8.3 Arborescence USB

```bash
lsusb -t
```

**Sortie** :
```
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
        |__ Port 4: Dev 3, If 0, Class=Video, Driver=uvcvideo, 480M
        |__ Port 4: Dev 3, If 1, Class=Video, Driver=uvcvideo, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=uhci_hcd/2p, 12M
    |__ Port 2: Dev 2, If 0, Class=Wireless, Driver=btusb, 12M
```

**Explication** :
- **Bus 01** : Contrôleur USB 2.0 (480M)
  - Dev 2 : Hub interne (6 ports)
    - Dev 3 : Webcam (2 interfaces : vidéo + contrôle)
- **Bus 02** : Contrôleur USB 1.1 (12M)
  - Dev 2 : Bluetooth
- **Driver** : Module kernel utilisé
  - uvcvideo : USB Video Class (standard webcam)
  - btusb : Bluetooth USB
  - hub : Gestion de hub

### 8.4 Surveillance en temps réel

```bash
watch -n 0.5 lsusb
```

- Rafraîchit toutes les 0.5s
- Branchez/débranchez un périphérique
- Voyez l'apparition/disparition

**Logs kernel** :

```bash
sudo dmesg -w
```

**Puis branchez une clé USB** :
```
[12345.678901] usb 1-1.4: new high-speed USB device number 5 using ehci-pci
[12345.789012] usb 1-1.4: New USB device found, idVendor=0781, idProduct=5583
[12345.789023] usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[12345.789034] usb 1-1.4: Product: Ultra Fit
[12345.789045] usb 1-1.4: Manufacturer: SanDisk
[12345.789056] usb 1-1.4: SerialNumber: 0123456789ABCDEF
[12345.901234] usb-storage 1-1.4:1.0: USB Mass Storage device detected
[12346.012345] scsi host6: usb-storage 1-1.4:1.0
[12347.123456] scsi 6:0:0:0: Direct-Access     SanDisk  Ultra Fit        1.00 PQ: 0 ANSI: 6
[12347.234567] sd 6:0:0:0: [sdb] 30310400 512-byte logical blocks: (15.5 GB/14.4 GiB)
[12347.345678] sd 6:0:0:0: [sdb] Write Protect is off
[12347.456789] sd 6:0:0:0: [sdb] Write cache: disabled
[12347.567890]  sdb: sdb1
[12347.678901] sd 6:0:0:0: [sdb] Attached SCSI removable disk
```

**Analyse** :
1. **new high-speed USB device** : Détecté en USB 2.0
2. **idVendor=0781, idProduct=5583** : Identifiants
3. **Product: Ultra Fit** : Nom du produit
4. **USB Mass Storage** : Type de périphérique
5. **scsi host6** : Assigné au sous-système SCSI
6. **sd 6:0:0:0: [sdb]** : Apparaît comme /dev/sdb
7. **30310400 512-byte blocks** : Capacité (15.5 GB)
8. **sdb: sdb1** : Une partition détectée
9. **Attached SCSI removable disk** : Prêt à l'utilisation

### 8.5 Vitesse de transfert USB

#### Test avec dd

```bash
# ATTENTION : Remplacez /dev/sdX par votre périphérique USB !
# Ceci EFFACERA toutes les données !

# Vérifiez d'abord le bon périphérique
lsblk

# Test d'écriture (1GB de zéros)
sudo dd if=/dev/zero of=/dev/sdX bs=1M count=1024 oflag=direct

# Sortie exemple :
# 1024+0 records in
# 1024+0 records out
# 1073741824 bytes (1.1 GB) copied, 18.3456 s, 58.5 MB/s
```

**Vitesses attendues** :
- **USB 2.0** : 30-40 MB/s (pratique vs 60 MB/s théorique)
- **USB 3.0** : 100-400 MB/s (selon qualité de la clé)

**Si vitesse très basse (<10 MB/s sur USB 2.0)** :
- Clé USB de mauvaise qualité
- Problème de driver
- Port USB endommagé

### 8.6 Problèmes courants

#### Périphérique non reconnu

```bash
# Vérifier si détecté physiquement
lsusb

# Vérifier les logs
sudo dmesg | tail -20

# Forcer la réinitialisation du port USB
echo "1-1.4" | sudo tee /sys/bus/usb/drivers/usb/unbind
echo "1-1.4" | sudo tee /sys/bus/usb/drivers/usb/bind
```

#### Consommation électrique excessive

Certains périphériques consomment trop :

```bash
# Voir la consommation par port
cat /sys/bus/usb/devices/*/power/level
```

**Solution** : Hub USB alimenté externement

---

## 9. Architecture complète du système

### 9.1 Vue d'ensemble matérielle

```bash
sudo lshw -short
```

**Sortie (simplifiée)** :
```
H/W path          Device      Class          Description
======================================================
                              system         20AT004FGE (LENOVO_MT_20AT)
/0                            bus            20AT004FGE
/0/0                          memory         64KiB BIOS
/0/4                          processor      Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
/0/4/5                        memory         64KiB L1 cache
/0/4/6                        memory         512KiB L2 cache
/0/4/7                        memory         3MiB L3 cache
/0/d                          memory         8GiB System Memory
/0/d/0                        memory         4GiB SODIMM DDR3 Synchronous 1333 MHz
/0/d/1                        memory         4GiB SODIMM DDR3 Synchronous 1333 MHz
/0/100                        bridge         2nd Generation Core Processor Family DRAM Controller
/0/100/2                      display        2nd Generation Core Processor Family Integrated Graphics Controller
/0/100/16                     communication  6 Series/C200 Series Chipset Family MEI Controller #1
/0/100/19        enp0s25     network        82579LM Gigabit Network Connection
/0/100/1a                     bus            6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2
/0/100/1b                     multimedia     6 Series/C200 Series Chipset Family High Definition Audio Controller
/0/100/1c                     bridge         6 Series/C200 Series Chipset Family PCI Express Root Port 1
/0/100/1c/0      wlp3s0      network        Centrino Advanced-N 6205 [Taylor Peak]
/0/100/1f                     bridge         QM67 Express Chipset LPC Controller
/0/100/1f.2                   storage        6 Series/C200 Series Chipset Family 6 port Mobile SATA AHCI Controller
/0/100/1f.3                   bus            6 Series/C200 Series Chipset Family SMBus Controller
/0/1             scsi0       storage        
/0/1/0.0.0       /dev/sda    disk           250GB Samsung SSD 850
/0/1/0.0.0/1     /dev/sda1   volume         511MiB EXT4 volume
/0/1/0.0.0/2     /dev/sda2   volume         97GiB EXT4 volume
/0/1/0.0.0/3     /dev/sda3   volume         7812MiB Linux swap volume
```

**Hiérarchie** :
- **/0** : Carte mère (bus principal)
  - /0/0 : BIOS
  - /0/4 : CPU + caches
  - /0/d : RAM (2 slots)
  - /0/100 : Northbridge (contrôleur mémoire/PCI)
    - /0/100/2 : GPU
    - /0/100/19 : Ethernet
    - /0/100/1a : Contrôleur USB
    - /0/100/1c/0 : WiFi (sur bus PCIe)
    - /0/100/1f : Southbridge (périphériques lents)
      - /0/100/1f.2 : Contrôleur SATA
  - /0/1 : Stockage SCSI (disque)

### 9.2 DMI / SMBIOS

```bash
sudo dmidecode
```

**Informations BIOS** :
```
BIOS Information
        Vendor: LENOVO
        Version: GJET97WW (2.47 )
        Release Date: 09/13/2018
        ROM Size: 8192 kB
```

**Carte mère** :
```
Base Board Information
        Manufacturer: LENOVO
        Product Name: 20AT004FGE
        Version: Not Defined
        Serial Number: 1ZXXXX123456
```

**Processeur** :
```
Processor Information
        Socket Designation: CPU Socket - U3E1
        Type: Central Processor
        Family: Core i5
        Manufacturer: Intel(R) Corporation
        ID: A7 06 02 00 FF FB EB BF
        Version: Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
        Voltage: 1.2 V
        Max Speed: 2500 MHz
        Current Speed: 2500 MHz
        Status: Populated, Enabled
        Upgrade: Socket rPGA988B
        L1 Cache Handle: 0x0005
        L2 Cache Handle: 0x0006
        L3 Cache Handle: 0x0007
```

### 9.3 Modules kernel chargés

```bash
lsmod | head -20
```

**Sortie** :
```
Module                  Size  Used by
snd_hda_codec_hdmi     53248  1
snd_hda_codec_realtek 110592  1
iwlmvm                405504  0
mac80211              864256  1 iwlmvm
iwlwifi               282624  1 iwlmvm
e1000e                278528  0
uvcvideo               94208  0
videobuf2_vmalloc      16384  1 uvcvideo
i915                 2232320  3
drm_kms_helper        184320  1 i915
```

**Modules importants** :
- **iwlwifi / iwlmvm** : WiFi Intel
- **e1000e** : Ethernet Intel
- **i915** : GPU Intel
- **uvcvideo** : Webcam USB
- **snd_hda_\*** : Audio

**Informations sur un module** :

```bash
modinfo i915
```

**Sortie** :
```
filename:       /lib/modules/5.14.0/kernel/drivers/gpu/drm/i915/i915.ko.xz
license:        GPL and additional rights
description:    Intel Graphics
author:         Intel Corporation
firmware:       i915/tgl_dmc_ver2_12.bin
...
parm:           modeset:Use kernel modesetting [KMS] (0=disable, 1=on, -1=force vga console preference) (int)
```

- **filename** : Chemin du module
- **description** : Rôle
- **firmware** : Fichiers firmware nécessaires
- **parm** : Paramètres configurables

**Charger/décharger un module** :

```bash
# Charger
sudo modprobe <nom_module>

# Décharger (si non utilisé)
sudo modprobe -r <nom_module>
```

### 9.4 Interruptions (IRQ)

```bash
cat /proc/interrupts
```

**Sortie (extraits)** :
```
           CPU0       CPU1       CPU2       CPU3       
  0:         42          0          0          0   IO-APIC   2-edge      timer
  1:       9876       1234       5678       2345   IO-APIC   1-edge      i8042
  8:          0          0          1          0   IO-APIC   8-edge      rtc0
  9:          0          0          0          0   IO-APIC   9-fasteoi   acpi
 16:      12345      23456      34567      45678   IO-APIC  16-fasteoi   i915
 17:     123456     234567     345678     456789   IO-APIC  17-fasteoi   snd_hda_intel
 19:      98765      87654      76543      65432   IO-APIC  19-fasteoi   enp0s25
```

**IRQ (Interrupt Request)** :
- Ligne de communication entre périphérique et CPU
- Le périphérique "interrompt" le CPU pour signaler un événement

**Colonnes** :
- **0-3** : Nombre d'interruptions par CPU
- **IO-APIC** : Contrôleur d'interruptions (moderne)
- **edge / fasteoi** : Type de déclenchement
- **Nom** : Périphérique concerné

**Périphériques** :
- **timer** : Horloge système (nombreuses interruptions)
- **i8042** : Contrôleur clavier/souris
- **i915** : GPU (beaucoup d'interruptions si affichage actif)
- **enp0s25** : Ethernet (interruptions lors de paquets reçus)

**Problème : IRQ partagées** :

Anciens systèmes partageaient les IRQ entre plusieurs périphériques, causant des conflits. Systèmes modernes (MSI) évitent ce problème.

**Si trop d'interruptions sur un périphérique** :
- Driver bogué
- Périphérique défectueux
- Boucle d'interruptions (exemple : câble réseau défectueux)

### 9.5 Ports I/O et adresses mémoire

```bash
cat /proc/ioports
```

**Sortie (extraits)** :
```
0000-0cf7 : PCI Bus 0000:00
  0000-001f : dma1
  0020-0021 : pic1
  0040-0043 : timer0
  0050-0053 : timer1
  0060-0060 : keyboard
  0064-0064 : keyboard
  0070-0071 : rtc0
5000-503f : 0000:00:02.0
  5000-503f : i915
5080-509f : 0000:00:19.0
  5080-509f : e1000e
```

**I/O Ports** :
- Adresses mémoire spéciales pour communiquer avec le matériel
- Architecture x86 traditionnelle (vs memory-mapped I/O)

**Exemples** :
- **0060-0064** : Clavier (port PS/2 historique)
- **5000-503f** : GPU Intel i915
- **5080-509f** : Carte Ethernet e1000e

```bash
cat /proc/iomem | head -30
```

**Sortie** :
```
00000000-00000fff : Reserved
00001000-0009fbff : System RAM
000a0000-000bffff : PCI Bus 0000:00
000c0000-000cffff : Video ROM
000f0000-000fffff : Reserved
  000f0000-000fffff : System ROM
00100000-bf79efff : System RAM
e0000000-efffffff : PCI Bus 0000:00
  e0000000-efffffff : 0000:00:02.0
    e0000000-efffffff : i915
f0000000-f03fffff : 0000:00:02.0
  f0000000-f03fffff : i915
```

**Zones mémoire** :
- **System RAM** : Votre RAM physique
- **Video ROM** : BIOS de la carte graphique
- **PCI Bus** : Espace mémoire alloué aux périphériques PCIe
- **i915** : Mémoire mappée du GPU

### 9.6 DMA (Direct Memory Access)

```bash
cat /proc/dma
```

**Sortie** :
```
 4: cascade
```

**DMA** permet aux périphériques d'accéder directement à la RAM sans passer par le CPU :
- **Plus rapide** : CPU libre pour autres tâches
- **Essentiel** pour audio, vidéo, disques

**cascade** : Canal DMA 4 utilisé pour chaîner les contrôleurs DMA

Sur systèmes modernes, DMA évolué vers **Bus Mastering** (périphériques PCI contrôlent entièrement leurs transferts).
---

## 10. Méthodologie de diagnostic

### 10.1 Approche systématique

Quand un composant ne fonctionne pas, suivez cette méthodologie :

#### Étape 1 : Identification du symptôme

**Questions à se poser** :
1. Quel est le problème exact ?
   - Le périphérique n'est pas détecté
   - Il est détecté mais ne fonctionne pas
   - Il fonctionne mais avec des erreurs
   - Performances dégradées

2. Quand le problème est-il apparu ?
   - Après une mise à jour système
   - Après une manipulation physique
   - Aléatoirement
   - Dès l'installation

3. Le problème est-il permanent ou intermittent ?

#### Étape 2 : Vérification de détection

```bash
# Vue d'ensemble
sudo lshw -short

# Spécifique selon le composant
lspci          # Périphériques PCI
lsusb          # Périphériques USB
lsblk          # Disques
ip link        # Cartes réseau
```

**Si le périphérique n'apparaît PAS** :
→ Problème matériel ou BIOS/UEFI
→ Allez à l'étape 3

**Si le périphérique apparaît** :
→ Problème de driver ou configuration
→ Allez à l'étape 4

#### Étape 3 : Vérification physique et BIOS

**Checklist physique** :
- [ ] Périphérique correctement installé (bien enfoncé)
- [ ] Alimentation branchée (si nécessaire)
- [ ] Pas de dommage visible
- [ ] Connecteurs propres (pas d'oxydation)
- [ ] LED d'activité fonctionnelle (si applicable)

**Checklist BIOS/UEFI** :
1. Accédez au BIOS (F1 au démarrage sur T420)
2. Vérifications :
   - [ ] Périphérique listé dans l'inventaire matériel
   - [ ] Périphérique activé (non disabled)
   - [ ] Mode de fonctionnement correct (ex: AHCI pour SATA)
   - [ ] Whitelist respectée (WiFi sur Lenovo)
   - [ ] Virtualisation activée (si utilisation de VMs)

**Test de substitution** :
- Testez le composant sur un autre PC (si possible)
- Testez un composant connu fonctionnel sur votre PC

#### Étape 4 : Vérification des drivers

```bash
# Lister les modules chargés
lsmod

# Chercher le driver spécifique
lsmod | grep -i <nom_composant>

# Exemples
lsmod | grep -i wifi
lsmod | grep -i e1000
lsmod | grep -i nvidia
```

**Si driver absent** :

**A) Identifier le driver nécessaire** :
```bash
# Trouver l'ID du périphérique
lspci -nn | grep -i <type>

# Exemple pour carte réseau
lspci -nn | grep -i network
# 03:00.0 Network controller [0280]: Intel Corporation Centrino Advanced-N 6205 [8086:0085] (rev 34)
```

Les chiffres entre crochets `[8086:0085]` sont Vendor:Product ID.

**B) Chercher le driver** :
```bash
# Rechercher dans la base de drivers
find /lib/modules/$(uname -r) -name "*8086*" -o -name "*0085*"

# Ou chercher en ligne
# "Intel 8086:0085 linux driver"
```

**C) Installer le driver** :
```bash
# Via gestionnaire de paquets (préféré)
sudo dnf search <nom_driver>
sudo dnf install <paquet>

# Exemple pour WiFi Broadcom
sudo dnf install broadcom-wl

# Ou compiler depuis les sources (avancé)
```

**D) Charger le driver** :
```bash
sudo modprobe <nom_module>

# Exemple
sudo modprobe iwlwifi
```

**E) Rendre permanent** :
```bash
# Créer un fichier de configuration
echo "iwlwifi" | sudo tee /etc/modules-load.d/wifi.conf
```

**Si driver présent mais ne fonctionne pas** :

**A) Vérifier les logs** :
```bash
# Logs généraux
sudo dmesg | grep -i <driver>
sudo journalctl -b | grep -i <driver>

# Logs en temps réel
sudo journalctl -f
```

**Recherchez** :
- `error` : Erreurs explicites
- `fail` : Échecs de chargement
- `timeout` : Problèmes de communication
- `firmware` : Firmware manquant

**B) Problème de firmware** :

De nombreux périphériques (WiFi, GPU) nécessitent un **firmware** :
- Petit programme chargé dans le périphérique
- Stocké dans `/lib/firmware/`

```bash
# Vérifier les firmwares installés
ls /lib/firmware/ | grep -i <fabricant>

# Exemple Intel WiFi
ls /lib/firmware/ | grep iwlwifi
```

**Si firmware manquant** :
```bash
# Installer le paquet de firmwares
sudo dnf install linux-firmware

# Ou paquet spécifique
sudo dnf install iwl7260-firmware
```

**C) Paramètres du module** :

Certains drivers ont des options configurables :

```bash
# Voir les options disponibles
modinfo <module> | grep ^parm

# Exemple
modinfo iwlwifi | grep ^parm
# parm:           swcrypto:using crypto in software (default 0 [hardware]) (int)
# parm:           11n_disable:disable 11n functionality (default 0 [on]) (int)
```

**Modifier les paramètres** :
```bash
# Temporaire (jusqu'au reboot)
sudo modprobe -r iwlwifi
sudo modprobe iwlwifi 11n_disable=1

# Permanent
echo "options iwlwifi 11n_disable=1" | sudo tee /etc/modprobe.d/iwlwifi.conf
```

#### Étape 5 : Tests de fonctionnement

**RAM** :
```bash
free -h                    # Capacité reconnue
sudo memtester 1G 1        # Test fonctionnel
```

**CPU** :
```bash
lscpu                      # Spécifications
sensors                    # Température
stress --cpu 4 --timeout 60s  # Test de charge
```

**Disque** :
```bash
lsblk                      # Détection
sudo smartctl -H /dev/sda  # Santé
sudo hdparm -Tt /dev/sda   # Performance
```

**Réseau** :
```bash
ip link show               # État interface
ping -c 4 8.8.8.8         # Connectivité
sudo ethtool enp0s25       # Vitesse négociée
```

**USB** :
```bash
lsusb                      # Détection
# Brancher/débrancher et observer dmesg
```

#### Étape 6 : Analyse des performances

**Si composant détecté et fonctionnel mais lent** :

**A) Benchmarks de base** :
```bash
# CPU
sysbench cpu --threads=4 run

# Disque
sudo hdparm -Tt /dev/sda
sudo dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=direct

# RAM
sudo dd if=/dev/zero of=/dev/null bs=1M count=10240
```

**B) Comparaison avec valeurs attendues** :

Recherchez en ligne les performances typiques de votre matériel :
- "Intel i5-2520M benchmark"
- "Samsung 850 EVO read speed"
- Comparez vos résultats

**C) Identification du goulot d'étranglement** :

```bash
# Pendant une tâche lente, surveillez :

# Terminal 1 : charge CPU
top

# Terminal 2 : I/O disque
sudo iotop

# Terminal 3 : réseau
sudo nethogs

# Terminal 4 : mémoire
watch -n 1 free -h
```

**Goulot d'étranglement** identifié par :
- CPU : %CPU à 100%
- Disque : %wa (wait) élevé dans `top`
- RAM : swap utilisé
- Réseau : bande passante saturée

### 10.2 Cas pratiques de diagnostic

#### Cas 1 : WiFi ne fonctionne pas

**Symptôme** : Pas de réseau WiFi détecté

**Diagnostic** :
```bash
# 1. Vérifier si la carte est détectée
lspci | grep -i network

# Si rien : problème matériel ou BIOS
# Si présent : continuer
```

```bash
# 2. Vérifier le driver
lsmod | grep iwl

# Si absent : charger le driver
sudo modprobe iwlwifi
```

```bash
# 3. Vérifier l'état de l'interface
ip link show

# Si wlp3s0 absent : driver non chargé
# Si présent : continuer
```

```bash
# 4. Activer l'interface
sudo ip link set wlp3s0 up

# 5. Scanner les réseaux
sudo nmcli dev wifi list
```

**Solutions courantes** :

**A) Interrupteur matériel désactivé** :
- Vérifiez le bouton WiFi physique (sur le côté T420)
- Ou Fn+F5 pour activer

**B) Soft-block activé** :
```bash
rfkill list

# Si "Soft blocked: yes"
rfkill unblock wifi
```

**C) Firmware manquant** :
```bash
sudo dmesg | grep iwlwifi
# Cherchez "firmware: failed to load"

sudo dnf install linux-firmware
sudo reboot
```

#### Cas 2 : Disque non détecté

**Symptôme** : Nouveau disque invisible

**Diagnostic** :
```bash
# 1. Vérifier BIOS
# Au boot : F1 > Storage > SATA
# Vérifiez que le disque apparaît

# 2. Vérifier le mode SATA
# BIOS > SATA Mode : doit être AHCI (pas IDE)
```

```bash
# 3. Vérifier la détection système
lsblk
sudo fdisk -l

# Si absent : problème physique ou câble
# Si présent comme /dev/sdb : continuer
```

```bash
# 4. Vérifier s'il a une partition
sudo parted /dev/sdb print

# Si "unrecognised disk label"
# = disque neuf, besoin de partitionner
```

**Solutions** :

**A) Disque neuf - Créer une partition** :
```bash
# ATTENTION : ceci EFFACE le disque !

# Créer une table de partition GPT
sudo parted /dev/sdb mklabel gpt

# Créer une partition
sudo parted /dev/sdb mkpart primary ext4 1MiB 100%

# Formater
sudo mkfs.ext4 /dev/sdb1

# Monter
sudo mkdir /mnt/nouveau_disque
sudo mount /dev/sdb1 /mnt/nouveau_disque
```

**B) Montage automatique au boot** :
```bash
# Récupérer l'UUID
sudo blkid /dev/sdb1

# Éditer fstab
sudo nano /etc/fstab

# Ajouter :
UUID=xxxx-xxxx-xxxx-xxxx /mnt/nouveau_disque ext4 defaults 0 2

# Tester
sudo mount -a
```

**C) Problème de câble SATA** :
- Symptôme : disque apparaît/disparaît
- Dmesg montre : "exception Emask 0x0"
- Solution : remplacer le câble SATA

#### Cas 3 : Surchauffe CPU

**Symptôme** : Ralentissements, arrêts inopinés

**Diagnostic** :
```bash
# 1. Vérifier la température
sensors

# Si > 90°C : CRITIQUE
# Si 80-90°C : Problème de refroidissement
```

```bash
# 2. Vérifier le throttling
dmesg | grep -i "thermal\|throttle"

# Si "CPU thermal limit exceeded"
# = CPU réduit sa fréquence pour refroidir
```

```bash
# 3. Vérifier la vitesse du ventilateur
sensors | grep fan

# Si RPM très bas ou 0 : ventilateur bloqué/mort
```

**Solutions** :

**A) Nettoyer le système de refroidissement** :
1. Éteignez et ouvrez le T420
2. Localisez le ventilateur (coin gauche généralement)
3. Nettoyez avec air comprimé :
   - Grilles d'aération
   - Ailettes du radiateur
   - Pales du ventilateur

**B) Remplacer la pâte thermique** :

**Matériel nécessaire** :
- Pâte thermique (Arctic MX-4, Noctua NT-H1)
- Alcool isopropylique (>90%)
- Cotons-tiges
- Chiffon microfibre

**Procédure** :
1. Éteignez, retirez batterie et alimentation
2. Retirez le clavier (7 vis)
3. Dévissez le dissipateur thermique (5 vis numérotées)
4. Retirez le dissipateur **délicatement** (pâte colle)
5. Nettoyez l'ancienne pâte :
   - Sur le CPU (carré métallique)
   - Sur le dissipateur (base cuivre)
   - Utilisez alcool isopropylique
6. Appliquez nouvelle pâte :
   - Grain de riz au centre du CPU
   - NE PAS étaler (la pression du dissipateur le fera)
7. Revissez le dissipateur :
   - **IMPORTANT** : Serrez en ordre numérique (1→2→3→4→5)
   - Serrez en croix, progressivement
   - Ne forcez pas (risque de casser le die)

**C) Configurer le ventilateur avec thinkfan** :

```bash
# Installer
sudo dnf install thinkfan

# Détecter les capteurs
sudo sensors-detect

# Configurer
sudo nano /etc/thinkfan.conf
```

**Configuration exemple** :
```
# Capteurs de température
hwmon /sys/class/hwmon/hwmon0/temp1_input

# Contrôle ventilateur
fan /proc/acpi/ibm/fan

# Paliers de vitesse
# (température_min, température_max, vitesse)
(0,     0,      0)      # < 50°C : arrêt
(50,    55,     1)      # 50-55°C : vitesse 1
(55,    60,     2)      # 55-60°C : vitesse 2
(60,    65,     3)      # 60-65°C : vitesse 3
(65,    70,     5)      # 65-70°C : vitesse 5
(70,    75,     7)      # 70-75°C : vitesse 7 (max)
(75,    32767,  127)    # > 75°C : full speed (urgence)
```

```bash
# Activer
sudo systemctl enable --now thinkfan
```

#### Cas 4 : Écran noir au démarrage

**Symptôme** : PC démarre (ventilateur tourne) mais écran noir

**Diagnostic étape par étape** :

**Étape 1 : Distinguer boot complet vs échec de boot**
```bash
# Appuyez sur Ctrl+Alt+F2
# Si vous accédez à un terminal texte : boot OK, problème graphique
# Si rien : boot échoue
```

**Si terminal accessible** :
```bash
# 1. Connectez-vous
# 2. Vérifiez les logs
sudo journalctl -xb | grep -i "error\|fail"

# 3. Vérifiez le serveur X
systemctl status display-manager

# Si "failed" : problème de configuration graphique
```

**Solutions graphiques** :

**A) Driver propriétaire défaillant** :
```bash
# Repasser sur driver open-source
sudo dnf remove nvidia-*    # Si Nvidia
sudo reboot
```

**B) Résolution invalide** :
```bash
# Forcer mode sans échec
# Au GRUB : appuyez 'e' sur l'entrée Linux
# Ajoutez à la ligne "linux" : nomodeset
# Ctrl+X pour booter

# Une fois connecté, reconfigurez
```

**Si aucun terminal** :

**A) Tester la RAM** :
- Un module défectueux peut empêcher le boot
- Retirez un module, testez
- Inversez les modules entre slots

**B) Réinitialiser le BIOS** :
- Retirez la batterie CMOS (pile plate sur carte mère)
- Attendez 30 secondes
- Réinsérez

**C) Boot minimal** :
- Débranchez TOUS les périphériques USB
- Retirez le disque dur (bootez sur Live USB)
- Si boot OK : problème de disque ou GRUB

### 10.3 Outils de diagnostic avancés

#### stress-ng : Tests de stress complets

```bash
sudo dnf install stress-ng

# Stresser CPU, RAM, I/O simultanément
sudo stress-ng --cpu 4 --vm 2 --vm-bytes 1G --hdd 1 --timeout 60s

# Pendant le test, surveillez
watch -n 1 "sensors && free -h"
```

**Options utiles** :
- `--cpu N` : N workers CPU
- `--vm N` : N workers mémoire
- `--hdd N` : N workers disque
- `--io N` : N workers I/O
- `--matrix N` : Calculs matriciels (intense)

#### hwinfo : Informations exhaustives

```bash
sudo dnf install hwinfo

# Tout le matériel
sudo hwinfo --short

# Spécifique
sudo hwinfo --cpu
sudo hwinfo --disk
sudo hwinfo --network
```

**Avantage** : Plus détaillé que lshw, détecte certains périphériques invisibles ailleurs.

#### inxi : Résumé système

```bash
sudo dnf install inxi

# Résumé complet
inxi -F

# Avec options
inxi -Fxz    # F=complet, x=détails extra, z=masque infos sensibles
```

**Sortie typique** :
```
System:    Host: thinkpad Kernel: 5.14.0 x86_64 bits: 64
           Desktop: GNOME 40.10 Distro: AlmaLinux 8.9
Machine:   Type: Laptop System: LENOVO product: 20AT004FGE
           Mobo: LENOVO model: 20AT004FGE BIOS: LENOVO v: GJET97WW date: 09/13/2018
CPU:       Info: Dual Core model: Intel Core i5-2520M bits: 64 type: MT MCP
           Speed: 800 MHz min/max: 800/3200 MHz Core speeds (MHz): 1: 800 2: 1200 3: 800 4: 800
Graphics:  Device-1: Intel 2nd Generation Core Processor Family driver: i915 v: kernel
           Display: x11 server: X.Org 1.20.11 driver: loaded: modesetting resolution: 1366x768~60Hz
           OpenGL: renderer: Mesa DRI Intel HD Graphics 3000 v: 3.3 Mesa 21.2.6
Network:   Device-1: Intel 82579LM Gigabit Network driver: e1000e
           Device-2: Intel Centrino Advanced-N 6205 driver: iwlwifi
```

#### smartmontools : Monitoring disques avancé

```bash
# Test long (plusieurs heures)
sudo smartctl -t long /dev/sda

# Surveiller progression
sudo smartctl -c /dev/sda

# Après test, voir résultats
sudo smartctl -l selftest /dev/sda
sudo smartctl -a /dev/sda
```

**Surveillance continue** :
```bash
# Activer smartd daemon
sudo systemctl enable --now smartd

# Configurer /etc/smartd.conf
# Alertes email en cas de problème
```

### 10.4 Documentation et ressources

#### Logs système essentiels

**Emplacements** :
```bash
/var/log/messages          # Messages système généraux (RHEL/AlmaLinux)
/var/log/dmesg             # Messages kernel au boot
/var/log/Xorg.0.log        # Serveur graphique
```

**Avec systemd** (moderne) :
```bash
# Tous les logs
sudo journalctl

# Boot actuel
sudo journalctl -b

# Boot précédent (utile après crash)
sudo journalctl -b -1

# Logs en temps réel
sudo journalctl -f

# Depuis une date
sudo journalctl --since "2025-01-28 10:00:00"

# Pour un service
sudo journalctl -u NetworkManager

# Filtres combinés
sudo journalctl -b -p err    # Erreurs du boot actuel
```

**Priorités** :
- `emerg` : Système inutilisable
- `alert` : Action immédiate requise
- `crit` : Critique
- `err` : Erreur
- `warning` : Avertissement
- `notice` : Notable
- `info` : Informatif
- `debug` : Débogage

#### Recherche d'informations

**A) Identifier votre matériel exact** :
```bash
# Numéro de modèle complet
sudo dmidecode -s system-product-name
sudo dmidecode -s system-serial-number

# Numéro FRU (Lenovo)
sudo dmidecode -t 2 | grep "Serial Number"
```

**B) Ressources en ligne** :

**Documentation officielle** :
- AlmaLinux : https://wiki.almalinux.org/
- Kernel.org : https://www.kernel.org/doc/
- ThinkWiki : https://www.thinkwiki.org/wiki/Category:T420

**Drivers** :
- https://github.com/ (beaucoup de drivers open-source)
- Site du fabricant (Intel, Realtek, etc.)

**Forums** :
- https://www.reddit.com/r/thinkpad/
- https://forum.almalinux.org/
- https://unix.stackexchange.com/

**Bases de données matériel** :
- https://linux-hardware.org/ (probe votre système)
- https://www.linux-drivers.org/ (chercher par ID PCI)

**C) Probe votre système** :
```bash
# Installer hw-probe
sudo dnf install hw-probe

# Créer un rapport
sudo hw-probe -all -upload

# Vous recevez un URL à partager ou consulter
# Utile pour demander de l'aide en montrant votre config exacte
```
### 10.5 Prévention et maintenance

#### Checklist mensuelle

**Vérifications logicielles** :
```bash
# 1. Mises à jour système
sudo dnf update

# 2. Vérifier santé disque
sudo smartctl -H /dev/sda

# 3. Vérifier logs d'erreurs
sudo journalctl -p err --since "1 month ago"

# 4. Nettoyer logs anciens (si espace disque faible)
sudo journalctl --vacuum-time=30d

# 5. Vérifier espace disque
df -h

# 6. Chercher gros fichiers inutiles
sudo du -sh /var/* | sort -rh | head
```

**Vérifications matérielles** :
- [ ] Vérifier température sous charge
- [ ] Écouter bruits anormaux (ventilateur, disque)
- [ ] Tester batterie (capacité, temps d'utilisation)
- [ ] Nettoyer grilles d'aération (air comprimé)

#### Checklist annuelle

**Matériel** :
- [ ] Nettoyer intérieur (poussière)
- [ ] Re-appliquer pâte thermique (si > 3 ans)
- [ ] Vérifier état batterie
- [ ] Vérifier charnières écran
- [ ] Tester tous les ports (USB, audio, etc.)

**Logiciel** :
- [ ] Upgrade vers nouvelle version OS (si souhaité)
- [ ] Nettoyer paquets inutilisés
- [ ] Backup complet
- [ ] Vérifier santé SSD/HDD (SMART complet)

#### Bonnes pratiques

**Pour prolonger la vie du matériel** :

**Batterie** :
- Ne pas laisser toujours branchée à 100%
- Utiliser seuils de charge (Lenovo : 50-80%)
```bash
# ThinkPad : configurer seuils
echo 50 | sudo tee /sys/class/power_supply/BAT0/charge_start_threshold
echo 80 | sudo tee /sys/class/power_supply/BAT0/charge_stop_threshold
```

**SSD** :
- Activer TRIM automatique
```bash
# Vérifier si TRIM supporté
sudo hdparm -I /dev/sda | grep TRIM

# Activer TRIM hebdomadaire
sudo systemctl enable fstrim.timer
```

**Température** :
- Éviter surfaces molles (lit, canapé)
- Utiliser sur surface dure et plane
- Ventilation externe (support ventilé) si utilisation intensive

**Mise à jour BIOS/Firmware** :
- Vérifier version actuelle
```bash
sudo dmidecode -s bios-version
```
- Chercher mise à jour sur site Lenovo
- **ATTENTION** : Risqué, seulement si problème spécifique

---

## 11. Annexes

### 11.1 Glossaire

**ACPI** (Advanced Configuration and Power Interface) : Standard de gestion d'énergie

**AHCI** (Advanced Host Controller Interface) : Mode SATA moderne (vs IDE)

**APM** (Advanced Power Management) : Ancien système de gestion d'énergie

**BIOS** (Basic Input/Output System) : Firmware de la carte mère (ancien)

**Bus** : "Autoroute" de communication entre composants

**Cache** : Mémoire ultra-rapide proche du CPU (L1/L2/L3)

**CMOS** : Puce stockant les réglages BIOS (alimentée par pile)

**DMA** (Direct Memory Access) : Accès direct à la RAM par périphériques

**Driver** : Pilote, traducteur entre OS et matériel

**EDID** (Extended Display Identification Data) : Infos de l'écran

**EFI/UEFI** : Firmware moderne remplaçant le BIOS

**Firmware** : Logiciel embarqué dans le matériel

**GPIO** (General Purpose Input/Output) : Broches programmables

**HDD** : Disque dur mécanique

**IRQ** (Interrupt Request) : Ligne d'interruption matérielle

**Kernel** : Noyau du système d'exploitation

**Latence** : Délai de réponse

**Module** : Driver chargeable dans le kernel

**PCIe** (PCI Express) : Bus haute vitesse

**POST** (Power-On Self Test) : Auto-tests au démarrage

**SMART** : Système de surveillance disques

**SSD** : Disque à mémoire flash (sans mécanique)

**Throttling** : Réduction de fréquence (CPU/GPU) pour limiter température

**UEFI** : Voir EFI

**UUID** : Identifiant unique universel

### 11.2 Commandes de référence rapide

#### Informations système
```bash
lscpu                          # CPU
free -h                        # RAM
lsblk                          # Disques
ip link                        # Réseau
lspci                          # Périphériques PCI
lsusb                          # Périphériques USB
sudo lshw -short               # Vue d'ensemble
sudo dmidecode                 # BIOS/matériel
sensors                        # Températures
```

#### Monitoring
```bash
top                            # Processus (CPU/RAM)
htop                           # Version améliorée
iotop                          # I/O disque
nethogs                        # Bande passante réseau
watch -n 1 "sensors"           # Températures en temps réel
dmesg -w                       # Logs kernel en temps réel
```

#### Tests
```bash
stress --cpu 4 --timeout 60s   # Stress CPU
memtester 1G 1                 # Test RAM
sudo smartctl -t short /dev/sda # Test disque
ping -c 4 8.8.8.8              # Test réseau
```

#### Diagnostic
```bash
sudo journalctl -b -p err      # Erreurs du boot
dmesg | grep -i error          # Erreurs kernel
lsmod                          # Modules chargés
modinfo <module>               # Info module
sudo hwinfo --short            # Matériel détaillé
```

### 11.3 Arborescence /sys importante

```
/sys/
├── block/                  # Périphériques bloc (disques)
│   └── sda/
│       ├── size            # Taille en secteurs
│       └── stat            # Statistiques I/O
├── class/
│   ├── backlight/          # Luminosité écran
│   ├── net/                # Interfaces réseau
│   ├── power_supply/       # Batterie, alimentation
│   └── thermal/            # Gestion thermique
├── devices/                # Arbre hiérarchique des périphériques
├── firmware/
│   └── efi/                # Variables EFI
└── module/                 # Modules kernel chargés
```

### 11.4 Fichiers /proc importants

```
/proc/
├── cmdline                 # Ligne de commande du kernel
├── cpuinfo                 # Informations CPU
├── meminfo                 # Informations mémoire
├── modules                 # Modules chargés (= lsmod)
├── mounts                  # Systèmes de fichiers montés
├── version                 # Version du kernel
├── interrupts              # IRQ et statistiques
├── ioports                 # Ports I/O utilisés
├── iomem                   # Carte mémoire
├── bus/
│   ├── input/devices       # Périphériques d'entrée
│   └── pci/devices         # Périphériques PCI
└── sys/
    └── vm/                 # Paramètres mémoire virtuelle
```

### 11.5 Codes de sortie et debugging

**Codes de sortie communs** :
```bash
# Après une commande
echo $?

# 0   = Succès
# 1   = Erreur générale
# 2   = Mauvaise utilisation
# 126 = Commande non exécutable
# 127 = Commande non trouvée
# 130 = Terminé par Ctrl+C
# 137 = Processus tué (kill -9)
```

**Debugging de scripts** :
```bash
# Mode verbeux
bash -x script.sh

# Ou dans le script
#!/bin/bash
set -x    # Affiche chaque commande
set -e    # Arrête sur première erreur
set -u    # Erreur sur variable non définie
```

### 11.6 Raccourcis utiles

**Terminal** :
- `Ctrl+C` : Interrompre commande en cours
- `Ctrl+Z` : Suspendre (mettre en arrière-plan)
- `Ctrl+D` : EOF (fin de saisie) ou déconnexion
- `Ctrl+L` : Effacer l'écran (= `clear`)
- `Ctrl+R` : Recherche dans historique
- `!!` : Répéter dernière commande
- `sudo !!` : Répéter avec sudo

**Navigation** :
- `cd -` : Retour au dossier précédent
- `cd` : Retour au home
- `pushd` / `popd` : Pile de répertoires

**Historique** :
```bash
history                    # Voir historique
!123                       # Exécuter commande #123
!pi                        # Dernière commande commençant par "pi"
```

### 11.7 Configuration T420 optimale recommandée

**Matériel** :
- RAM : 2× 4GB DDR3 1600MHz (max supporté)
- Disque principal : SSD SATA 250-500GB (OS + apps)
- Disque secondaire : HDD 500GB-1TB dans Ultrabay (données)
- WiFi : Intel 6300 (tri-band, meilleur signal)

**BIOS** :
- Mode SATA : AHCI
- Virtualisation : Activée (si utilisation VMs)
- USB Boot : Activé
- Boot Mode : UEFI (si OS récent)

**Système** :
- OS : AlmaLinux 8 ou 9 (stabilité) ou Fedora (nouveautés)
- Kernel : Dernière version stable
- Firmwares : À jour (`linux-firmware`)

**Logiciel** :
```bash
# Outils de diagnostic essentiels
sudo dnf install \
  lm_sensors smartmontools htop iotop \
  nethogs sysstat pciutils usbutils \
  dmidecode ethtool hdparm stress \
  memtester hwinfo inxi
```

### 11.8 Checklist pre-upgrade matériel

Avant d'acheter un composant :

**Compatibilité** :
- [ ] Vérifier les spécifications exactes du T420
- [ ] Chercher "T420 upgrade <composant>" sur forums
- [ ] Vérifier whitelist BIOS (pour WiFi)
- [ ] Confirmer les connecteurs (DDR3 vs DDR4, SATA vs NVMe)

**Performances** :
- [ ] Le composant est-il réellement plus rapide ?
- [ ] Le système supporte-t-il sa vitesse max ?
- [ ] Quel gain réel attendu (benchmarks) ?

**Installation** :
- [ ] Ai-je les outils nécessaires ?
- [ ] Ai-je consulté un guide de démontage ?
- [ ] Ai-je prévu un backup ?

### 11.9 Points de vigilance spécifiques T420

**Whitelist WiFi** :
- Le BIOS Lenovo bloque les cartes WiFi non approuvées
- Solution : BIOS modifié (risqué) ou carte compatibles

**ExpressCard vs mSATA** :
- Slot ExpressCard peut accueillir adaptateur mSATA
- Gain : SSD supplémentaire rapide
- Attention : ExpressCard devient inutilisable

**Batterie** :
- Batterie 6 cellules (standard) ou 9 cellules (dépasse sous le laptop)
- Vérifier cycles avec `upower -i /org/freedesktop/UPower/devices/battery_BAT0`

**Upgrade écran** :
- Passage 1366×768 → 1600×900 possible
- Nécessite dalle compatible (même connecteur LVDS 40 pins)

---

## 12. Pour aller plus loin

### 12.1 Virtualisation et containers

Une fois votre matériel maîtrisé, explorez :

**KVM/QEMU** : Virtualisation matérielle
```bash
# Vérifier support
lscpu | grep Virtualization

# Si VT-x présent
sudo dnf groupinstall "Virtualization Host"
```

**Docker** : Containers légers
```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce
```

### 12.2 Monitoring avancé

**Prometheus + Grafana** : Monitoring professionnel
- Collecte métriques système
- Dashboards visuels
- Alertes personnalisées

**Netdata** : Monitoring temps réel
```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
# Interface web : http://localhost:19999
```

### 12.3 Overclocking (ATTENTION)

⚠️ **Risqué, non recommandé sur laptop**

**Undervolting CPU** (réduction consommation) :
```bash
# intel-undervolt (diminue voltage sans réduire fréquence)
# = Moins de chaleur, même performance
# Nécessite tests et prudence
```

### 12.4 Modifications matérielles avancées

**Coreboot** : BIOS open-source
- Remplace BIOS Lenovo
- Supprime whitelist
- Boot plus rapide
- **Risque de brick** (rendre inutilisable)

**Libreboot** : Version entièrement libre
- Pas de blobs propriétaires
- Compatibilité T420 limitée (modèles spécifiques)

### 12.5 Contribution à la communauté

**Documenter vos découvertes** :
- Créer guides sur ThinkWiki
- Partager configurations sur forums
- Contribuer à hw-probe database

**Développement** :
- Signaler bugs drivers (kernel bugzilla)
- Tester versions beta (kernels RC)
- Contribuer patches (si compétences)

---

## Conclusion

Vous avez maintenant une compréhension complète de :

✅ **Architecture matérielle** : CPU, RAM, disques, réseau, etc.
✅ **Outils de diagnostic** : lspci, lsusb, smartctl, sensors...
✅ **Méthodologie** : Approche systématique de résolution de problèmes
✅ **Installation physique** : Manipulation sécurisée des composants
✅ **Optimisation** : Améliorer performances et fiabilité

**Compétences acquises** :
- Identifier tout composant de votre système
- Diagnostiquer pannes matérielles
- Interpréter logs et métriques
- Effectuer upgrades matériels en sécurité
- Optimiser configuration système

**Pour devenir expert** :
1. **Pratiquez régulièrement** : Lancez `lscpu`, `sensors`, `lsblk` jusqu'à les lire naturellement
2. **Expérimentez** : Testez différents paramètres, drivers
3. **Cassez (en sécurité)** : VM ou vieux matériel pour apprendre sans risque
4. **Documentez** : Tenez un journal de vos interventions
5. **Partagez** : Aidez d'autres sur forums, renforcez votre apprentissage

**Ressources pour continuer** :
- Kernel documentation : /usr/share/doc/kernel-doc-*/Documentation/
- Man pages : `man lspci`, `man smartctl`, etc.
- ThinkWiki : https://www.thinkwiki.org/
- r/thinkpad et r/linuxhardware

**Le plus important** :
La curiosité et la méthodologie valent plus que la mémorisation. Avec ce guide comme référence et votre pratique régulière, vous maîtriserez parfaitement le diagnostic et la gestion matérielle de votre système Linux.
