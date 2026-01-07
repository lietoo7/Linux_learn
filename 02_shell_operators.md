# Fiche 2 : Shell Operators

Les opérateurs du shell sont des symboles spéciaux qui permettent de contrôler le flux de données, d'enchaîner des commandes et de gérer les résultats (succès ou échec).

## 1. Redirection des entrées et sorties (I/O Redirection)

Chaque commande Linux possède trois flux standards :

* **stdin (0)** : L'entrée standard (clavier).
* **stdout (1)** : La sortie standard (écran).
* **stderr (2)** : La sortie d'erreur (écran).

### Sortie (Output)

* `>` : Redirige la sortie standard vers un fichier. **Écrase** le fichier s'il existe.
* `ls > liste.txt`


* `>>` : Redirige la sortie standard en **ajoutant** à la fin du fichier sans l'écraser.
* `echo "Nouvelle ligne" >> liste.txt`


* `2>` : Redirige uniquement les **erreurs**.
* `ls /dossier/inexistant 2> erreurs.log`


* `&>` : Redirige à la fois la sortie standard et les erreurs vers le même fichier.

### Entrée (Input)

* `<` : Lit l'entrée d'une commande à partir d'un fichier plutôt que du clavier.
* `wc -l < liste.txt`



---

## 2. Le Pipe (Le tube) `|`

L'opérateur `|` est l'un des plus puissants sous Linux. Il connecte la **sortie** d'une commande directement à l'**entrée** de la suivante.

* **Concept :** `commande1 | commande2`
* **Exemple concret :** Trier les utilisateurs connectés par ordre alphabétique.
* `who | sort`


* **Exemple complexe :** Trouver les 5 fichiers les plus volumineux dans un dossier.
* `ls -S | head -n 5`



---

## 3. Enchaînement logique (Command Chaining)

Ces opérateurs permettent d'exécuter plusieurs commandes en une seule ligne selon le résultat des précédentes.

| Opérateur | Nom | Description |
| --- | --- | --- |
| `;` | Point-virgule | Exécute les commandes l'une après l'autre, peu importe le résultat. |
| `&&` | ET logique | Exécute la commande suivante **uniquement si** la précédente a réussi (exit code 0). |
| `||` | OU logique | Exécute la commande suivante **uniquement si** la précédente a échoué. |

**Exemples :**

* `mkdir test ; cd test` : Crée le dossier et y entre (même si la création échoue).
* `make && make install` : Compile le programme et l'installe seulement si la compilation est réussie.
* `ping -c 1 google.com || echo "Pas de réseau"` : Affiche un message d'alerte si le ping échoue.

---

## 4. Exécution en arrière-plan `&`

Placer un `&` à la fin d'une commande permet de la lancer en tâche de fond. Le terminal est immédiatement libéré pour d'autres commandes.

* **Usage :** `cp -r /gros/dossier /destination &`
* **Note :** Utilisez la commande `jobs` pour voir les tâches en cours et `fg` pour les ramener au premier plan.

---

## Résumé pratique

> **Astuce de pro :** Pour vider le contenu d'un fichier log sans le supprimer, on utilise souvent la redirection vide :
> `> application.log`
