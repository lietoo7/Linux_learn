# Fiche 1 : Interaction avec le système de fichiers

Cette fiche détaille les commandes fondamentales pour naviguer, visualiser et manipuler des fichiers et répertoires sous Linux.

## 1. Navigation et Localisation

### `pwd` (Print Working Directory)

Affiche le chemin absolu du répertoire dans lequel vous vous trouvez actuellement.

* **Usage :** `pwd`

### `ls` (List)

Liste le contenu d'un répertoire.

* `-l` : Format long (affiche permissions, taille, propriétaire).
* `-a` : Affiche tous les fichiers, y compris les fichiers cachés (commençant par un `.`).
* `-h` : Taille lisible par l'humain (Ko, Mo, Go).
* **Exemple :** `ls -lah`

---

## 2. Création et Manipulation

### `touch`

Crée un fichier vide ou met à jour l'horodatage d'un fichier existant.

* **Exemple :** `touch notes.txt`

### `mkdir` (Make Directory)

Crée un nouveau répertoire.

* `-p` : Crée les répertoires parents si nécessaire (ex: `mkdir -p projet/src/data`).

### `cp` (Copy)

Copie des fichiers ou des répertoires.

* `-r` : Copie récursive (obligatoire pour les dossiers).
* `-i` : Demande confirmation avant d'écraser.
* **Exemple :** `cp fichier.txt sauvegarde/`

### `mv` (Move)

Déplace ou renomme un fichier/répertoire.

* **Renommer :** `mv ancien_nom.txt nouveau_nom.txt`
* **Déplacer :** `mv dossier1/ /tmp/`

### `rm` (Remove)

Supprime des fichiers ou répertoires. **Attention : action irréversible.**

* `-r` : Supprime un répertoire et son contenu.
* `-f` : Force la suppression sans demander de confirmation.
* **Exemple :** `rm -rf dossier_inutile/`

---

## 3. Lecture et Analyse de contenu

### `cat` (Concatenate)

Affiche tout le contenu d'un fichier à l'écran.

* `-n` : Numérote les lignes.

### `file`

Indique le type de fichier (ASCII text, exécutable ELF, image PNG, etc.).

* **Usage :** `file mon_fichier`

### `wc` (Word Count)

Compte les éléments d'un fichier.

* `-l` : Compte les lignes.
* `-w` : Compte les mots.
* **Exemple :** `wc -l log.txt`

---

## 4. Recherche et Filtrage

### `grep` (Global Regular Expression Print)

Recherche une chaîne de caractères dans un fichier ou un flux.

* `-i` : Ignore la casse (majuscules/minuscules).
* `-v` : Inverse la recherche (affiche ce qui ne correspond *pas*).
* `-r` : Recherche récursive dans les dossiers.
* **Exemple :** `grep "error" /var/log/syslog`

### `find`

Recherche des fichiers dans la hiérarchie selon des critères précis.

* `-name` : Recherche par nom.
* `-type f` ou `-type d` : Recherche uniquement les fichiers (f) ou dossiers (d).
* `-size` : Recherche par taille (ex: `+100M`).
* **Exemple :** `find /home -name "*.pdf"`

---

## Résumé pratique

> **Scénario :** Chercher tous les fichiers ".log" dans `/var`, compter combien contiennent le mot "CRITICAL".
> `find /var -name "*.log" | xargs grep "CRITICAL" | wc -l`
