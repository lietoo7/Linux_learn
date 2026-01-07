# Fiche 3 : Permissions 101

Sous Linux, tout est fichier. La sécurité repose sur un système de droits appliqués à trois catégories d'utilisateurs, pour trois types d'actions.

## 1. Les trois catégories d'utilisateurs

Lorsqu'on regarde les propriétés d'un fichier (via `ls -l`), on distingue :

* **u** (User/Owner) : L'utilisateur qui possède le fichier.
* **g** (Group) : Les membres du groupe associé au fichier.
* **o** (Others) : Tous les autres utilisateurs du système.

## 2. Les types de droits (rwx)

| Symbole | Action | Valeur Octale |
| --- | --- | --- |
| **r** (read) | Lire le contenu d'un fichier / Lister un dossier. | **4** |
| **w** (write) | Modifier le fichier / Créer ou supprimer dans un dossier. | **2** |
| **x** (execute) | Lancer un programme / Entrer dans un dossier (`cd`). | **1** |

---

## 3. Comprendre la sortie de `ls -l`

Si vous tapez `ls -l`, vous verrez une chaîne de 10 caractères comme celle-ci :

`-rwxr-xr--`

1. **Le 1er caractère** : Type de fichier (`-` pour fichier, `d` pour dossier).
2. **Les 3 suivants (u)** : `rwx` (le propriétaire peut tout faire).
3. **Les 3 suivants (g)** : `r-x` (le groupe peut lire et exécuter).
4. **Les 3 derniers (o)** : `r--` (les autres peuvent seulement lire).

---

## 4. Modifier les permissions : `chmod`

Il existe deux méthodes pour utiliser `chmod` (Change Mode).

### La méthode numérique (Octale)

On additionne les valeurs (4, 2, 1) pour chaque catégorie.

* **7** (4+2+1) : Tous les droits.
* **6** (4+2) : Lecture + Écriture.
* **5** (4+1) : Lecture + Exécution.
* **4** : Lecture seule.

**Exemple :**
`chmod 754 mon_fichier`
*(Propriétaire: 7, Groupe: 5, Autres: 4)*

### La méthode symbolique

On utilise les lettres et les opérateurs `+`, `-`, `=`.

* `chmod u+x script.sh` : Ajoute le droit d'exécution au propriétaire.
* `chmod g-w notes.txt` : Retire le droit d'écriture au groupe.
* `chmod o=r rapport.pdf` : Définit les droits des "autres" sur lecture seule uniquement.

---

## 5. Changer le propriétaire : `chown`

Seul l'utilisateur **root** (ou via `sudo`) peut changer le propriétaire d'un fichier.

* **Changer le propriétaire :** `sudo chown alice fichier.txt`
* **Changer le propriétaire et le groupe :** `sudo chown alice:admins fichier.txt`
* **Changer récursivement (pour un dossier) :** `sudo chown -R alice:alice dossier/`

---

## Résumé pratique

> **Attention au dossier :** Pour qu'un utilisateur puisse faire un `cd` dans un dossier, il doit impérativement avoir le droit **x** (exécution) sur ce dossier. Le droit **r** seul permet de voir la liste des fichiers, mais pas d'y accéder.
 
