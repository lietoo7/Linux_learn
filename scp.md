La commande **`scp`** (Secure Copy) est l'outil standard pour transférer des fichiers ou des répertoires de manière sécurisée entre deux machines via le protocole SSH.

 
---

## 1. Syntaxe de base

La structure suit toujours la logique : **source** vers **destination**.

```bash
scp [options] [source] [destination]

```

### Cas pratiques :

* **Copier du local vers le distant :**
`scp fichier.txt utilisateur@ip_distante:/home/utilisateur/destination/`
* **Copier du distant vers le local :**
`scp utilisateur@ip_distante:/home/utilisateur/fichier_distant.txt /chemin/local/`

---

## 2. Les options indispensables

Voici les drapeaux que vous utiliserez 90% du temps :

* **`-r` (Recursive) :** Indispensable pour copier un dossier entier et tout son contenu.
* **`-P [port]` :** Pour spécifier un port SSH différent du port 22 (attention, c'est un **P majuscule**).
* **`-i [identity_file]` :** Pour utiliser une clé SSH spécifique (private key) au lieu d'un mot de passe.
* **`-C` (Compression) :** Compresse les données pendant le transfert, utile pour les gros fichiers sur des connexions lentes.
* **`-q` (Quiet) :** Désactive la barre de progression (utile dans les scripts).

---

## 3. Exemples concrets

### Transférer un dossier complet vers un serveur

```bash
scp -r ./mon_dossier_outils/ admin@192.168.1.50:/tmp/

```

### Récupérer un fichier de config en utilisant une clé SSH

```bash
scp -i ~/.ssh/id_rsa root@10.0.0.5:/etc/shadow ./backup_shadow

```

### Transférer entre deux serveurs distants (depuis votre machine)

```bash
scp user1@server1:/chemin/fichier.txt user2@server2:/chemin/destination/

```

---

## 4. Points d'attention (Sécurité & Usage)

> [!IMPORTANT]
> **Écrasement :** `scp` écrase les fichiers de destination de même nom sans demander de confirmation.
> **Remplacement par `rsync` :** Bien que `scp` soit historique, beaucoup d'administrateurs préfèrent aujourd'hui `rsync`. Pourquoi ? Parce que `rsync` permet de reprendre un transfert interrompu et ne transfère que les modifications, ce qui est beaucoup plus efficace.
 
