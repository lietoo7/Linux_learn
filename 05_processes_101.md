# Fiche 5 : Processes 101

Un **processus** est une instance d'un programme en cours d'exécution. Chaque processus possède un identifiant unique appelé **PID** (*Process ID*).

## 1. Visualiser les processus

### `ps` (Process Status)

Affiche un instantané des processus actuels.

* `ps aux` : La commande classique pour tout voir.
* `a` : Tous les utilisateurs.
* `u` : Affiche l'utilisateur (owner) et l'utilisation RAM/CPU.
* `x` : Inclut les processus sans terminal (services).



### `top` & `htop`

Affiche une vue dynamique et interactive.

* `top` : L'outil standard pré-installé.
* `htop` : Plus moderne, coloré et permet de naviguer avec les flèches (souvent à installer via `apt install htop`).

---

## 2. Cycle de vie et Hiérarchie

* **Parent & Child** : Presque chaque processus est créé par un autre processus (le parent). Le premier processus lancé au démarrage est généralement `systemd` (PID 1).
* **PPID** : *Parent Process ID*.

---

## 3. Terminer un processus : `kill`

Si un programme ne répond plus, vous pouvez lui envoyer des "signaux" pour l'arrêter.

* `kill <PID>` : Envoie le signal **SIGTERM** (15). Demande gentiment au programme de s'arrêter en sauvegardant ses données.
* `kill -9 <PID>` : Envoie le signal **SIGKILL** (9). Force l'arrêt immédiat (brutal).
* `killall <nom_du_programme>` : Tue tous les processus portant ce nom (ex: `killall firefox`).

---

## 4. Gestion en arrière-plan (Jobs)

Parfois, vous voulez lancer une commande tout en gardant l'usage de votre terminal.

* **Lancer en arrière-plan** : Ajouter `&` à la fin.
* `sleep 100 &`


* **Suspendre un processus** : `Ctrl + Z`. Cela met le processus en pause.
* **Reprendre en arrière-plan** : `bg` (*background*).
* **Reprendre au premier plan** : `fg` (*foreground*).
* **Lister les tâches du shell** : `jobs`.

---

## 5. Priorité des processus : Nice

Le noyau Linux décide du temps CPU accordé à chaque processus. On peut influencer cela avec la valeur "Nice" (de -20 à 19).

* Plus la valeur est **élevée** (ex: 19), plus le processus est "gentil" (il laisse la priorité aux autres).
* Plus la valeur est **basse** (ex: -10), plus il est prioritaire.

---

## Résumé pratique

> **Comment arrêter un processus gourmand ?**
> 1. Trouver le PID avec `top` ou `ps aux | grep nom_du_prog`.
> 2. Tenter un arrêt propre : `kill <PID>`.
> 3. Si ça résiste : `kill -9 <PID>`.
 
