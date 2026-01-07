# Fiche 6 : Automation

L'automatisation sous Linux repose sur deux piliers : le **Scripting Bash** (pour enchaîner les actions) et le **Cron** (pour planifier l'exécution).

## 1. Scripting Bash : Les Bases

Un script est un fichier texte contenant une suite de commandes.

### La structure minimale

1. **Le Shebang** : La première ligne `#!/bin/bash` indique au système qu'il doit utiliser l'interpréteur Bash.
2. **Les Permissions** : Un script doit avoir le droit d'exécution (`chmod +x script.sh`).

**Exemple de script (`backup.sh`) :**

```bash
#!/bin/bash
# Ceci est un commentaire
echo "Début de la sauvegarde..."
tar -czf /home/user/backup_$(date +%Y%m%d).tar.gz /home/user/documents
echo "Sauvegarde terminée le $(date)"

```

### Variables et Arguments

* **Déclarer** : `NOM="Gemini"`
* **Utiliser** : `echo $NOM`
* **Arguments** : `$1` est le premier argument passé au script, `$2` le deuxième, etc.

---

## 2. Planification avec Cron

Le démon **crond** permet d'exécuter des scripts ou des commandes à des moments précis.

### La Crontab

Chaque utilisateur a sa propre table de planification. On y accède avec :

* `crontab -e` : Modifier la table.
* `crontab -l` : Lister les tâches prévues.

### Syntaxe de Cron

Une ligne de crontab se compose de 5 champs de temps suivis de la commande :

```text
.---------------- minute (0 - 59)
|  .------------- heure (0 - 23)
|  |  .---------- jour du mois (1 - 31)
|  |  |  .------- mois (1 - 12)
|  |  |  |  .---- jour de la semaine (0 - 6) (dimanche est 0 ou 7)
|  |  |  |  |
* * * * * commande à exécuter

```

**Exemples :**

* `0 5 * * * /home/user/backup.sh` : Lance le script tous les jours à 5h00 du matin.
* `*/15 * * * * command` : Lance la commande toutes les 15 minutes.
* `0 12 * * 1 command` : Lance la commande tous les lundis à midi pile.

---

## 3. Automatisation avancée

### Les boucles (For loops)

Utile pour traiter plusieurs fichiers d'un coup.

```bash
for fichier in *.txt; do
    mv "$fichier" "${fichier%.txt}.bak"
done

```

### Les conditions (If statements)

```bash
if [ -f "/etc/passwd" ]; then
    echo "Le fichier existe."
fi

```

---

## Résumé pratique

> **Checklist pour un script qui fonctionne :**
> 1. Commencer par `#!/bin/bash`.
> 2. Enregistrer et quitter.
> 3. Rendre exécutable : `chmod +x mon_script.sh`.
> 4. Tester manuellement : `./mon_script.sh`.
> 5. L'ajouter à la `crontab` si besoin d'automatisation temporelle.
 
