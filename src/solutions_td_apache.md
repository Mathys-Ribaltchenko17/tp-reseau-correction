# Solutions du TD Architecture des réseaux : Serveur Web Apache

*Note : Dans toutes les commandes et configurations ci-dessous, veillez à remplacer le `X` par le numéro de votre machine virtuelle (de 1 à 35).*

---

## Préambule : Configuration initiale

Avant de configurer Apache, il faut s'assurer que la machine virtuelle (VM) dispose de la bonne configuration réseau et des bons paquets.

**1. Configuration des interfaces réseaux (IPs)**
Votre VM doit écouter sur deux adresses IP pour l'interface `eth0`. Vous pouvez les ajouter temporairement avec les commandes suivantes :
```bash
ip addr add 192.168.X.1/24 dev eth0
ip addr add 192.168.X.2/24 dev eth0
```

**2. Installation d'Apache**
```bash
apt update
apt install apache2
```

**3. Configuration du DNS et de la route**
On modifie le fichier `/etc/resolv.conf` pour qu'il ne contienne que le serveur DNS de l'IUT (zone `fai.com`), puis on ajoute la route pour le joindre :
```bash
echo "nameserver 192.168.0.254" > /etc/resolv.conf
ip route add 192.168.0.0/16 dev eth0
```

**4. Tests de validation**
```bash
ping -c 3 192.168.0.254
host wwwX.fai.com
host wwX.fai.com
host wX.fai.com
```

---

## 3. Réalisations
### 3.1 Premier site

#### Site par défaut

L'objectif est de remplacer le site par défaut d'Apache par le vôtre (sur `wwwX.fai.com`).

**1. Désactiver le site initial d'Apache :**
```bash
a2dissite 000-default.conf
```

**2. Télécharger et extraire l'archive :**
```bash
wget https://gitlab.univ-nantes.fr/iut.info2.r3_06/r3_06.tp1/-/raw/main/r3_06.tp1.tgz --output-document=- | tar xz -C /var/www/html
```

**3. Création du fichier de configuration du site :**
Créez et éditez le fichier `/etc/apache2/sites-available/web.conf` :
```bash
nano /etc/apache2/sites-available/web.conf
```

Insérez la configuration suivante :
```apache
<VirtualHost *:80>
    ServerName wwwX.fai.com
    ServerAdmin webmasterX@fai.com
    DocumentRoot /var/www/html/web

    # Autoriser l'accès au répertoire racine de ce site
    <Directory /var/www/html/web>
        Require all granted
    </Directory>
</VirtualHost>
```

**4. Activer le site et relancer Apache :**
```bash
a2ensite web.conf
systemctl reload apache2
```

#### Fichier par défaut / Gestion des permissions

On ajoute des règles spécifiques pour les sous-répertoires `listing`, `bizarre` et `rien`. Modifiez le fichier `/etc/apache2/sites-available/web.conf` pour l'enrichir ainsi :

```apache
<VirtualHost *:80>
    ServerName wwwX.fai.com
    ServerAdmin webmasterX@fai.com
    DocumentRoot /var/www/html/web

    <Directory /var/www/html/web>
        Require all granted
    </Directory>

    # 1. Listing toujours affiché pour le répertoire "listing"
    <Directory /var/www/html/web/listing>
        Options +Indexes
    </Directory>

    # 2. Pas de listing pour "bizarre" et default.html prioritaire
    <Directory /var/www/html/web/bizarre>
        Options -Indexes
        DirectoryIndex default.html index.html
    </Directory>

    # 3. Pas de listing ni de contenu par défaut pour "rien"
    <Directory /var/www/html/web/rien>
        Options -Indexes
        DirectoryIndex disabled
    </Directory>
</VirtualHost>
```
*Explications :*
* `Options +Indexes` force l'affichage de l'arborescence s'il n'y a pas de fichier index.
* `Options -Indexes` interdit ce listage (renverra une erreur 403 si aucun fichier par défaut n'est trouvé).
* `DirectoryIndex default.html index.html` indique à Apache de chercher d'abord `default.html`, puis `index.html`.
* `DirectoryIndex disabled` désactive totalement la recherche de fichier par défaut. Associé à `-Indexes`, cela verrouille l'affichage (erreur 403 garantie).

Pensez à recharger la configuration : `systemctl reload apache2`.

---

### 3.2 Serveurs virtuels

Pour ces questions, on demande des journaux (logs) différents pour chaque VirtualHost.

#### Serveur virtuel par nom (`wX.fai.com`)

Créez un fichier `/etc/apache2/sites-available/web2.conf` :
```apache
<VirtualHost *:80>
    ServerName wX.fai.com
    DocumentRoot /var/www/html/web2

    <Directory /var/www/html/web2>
        Require all granted
    </Directory>

    # Fichiers de logs distincts pour ce site
    ErrorLog ${APACHE_LOG_DIR}/web2_error.log
    CustomLog ${APACHE_LOG_DIR}/web2_access.log combined
</VirtualHost>
```
Activez-le : `a2ensite web2.conf` puis `systemctl reload apache2`.

#### Serveur virtuel par adresse (`wwX.fai.com` sur IP .2)

Créez un fichier `/etc/apache2/sites-available/web3.conf` :
```apache
# Écoute spécifiquement sur la 2ème IP
<VirtualHost 192.168.X.2:80>
    ServerName wwX.fai.com
    DocumentRoot /var/www/html/web3

    <Directory /var/www/html/web3>
        Require all granted
    </Directory>

    # Fichiers de logs distincts
    ErrorLog ${APACHE_LOG_DIR}/web3_error.log
    CustomLog ${APACHE_LOG_DIR}/web3_access.log combined
</VirtualHost>
```
Activez-le : `a2ensite web3.conf` puis `systemctl reload apache2`.

#### Serveur virtuel par port (Subsidiaire : port 8080)

**1. Modifier les ports d'écoute :**
Éditez `/etc/apache2/ports.conf` et ajoutez la ligne `Listen 8080` sous `Listen 80`.

**2. Créer le site web4.conf :**
Créez `/etc/apache2/sites-available/web4.conf` :
```apache
<VirtualHost 192.168.X.1:8080>
    ServerName wwwX.fai.com
    DocumentRoot /var/www/html/web4

    <Directory /var/www/html/web4>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/web4_error.log
    CustomLog ${APACHE_LOG_DIR}/web4_access.log combined
</VirtualHost>
```
Activez le site et relancez complètement Apache (car on a modifié les ports) :
```bash
a2ensite web4.conf
systemctl restart apache2
```

---

### 3.3 Répertoires virtuels

#### Cas général (Alias)

Nous voulons que `http://wwwX.fai.com/rep1/` pointe vers `/var/www/html/rep1` (qui est en dehors du `DocumentRoot` de `wwwX.fai.com`).

Modifiez votre fichier `/etc/apache2/sites-available/web.conf` en y ajoutant, à l'intérieur de la balise `<VirtualHost *:80>`, les lignes suivantes :

```apache
    # Création de l'alias
    Alias "/rep1" "/var/www/html/rep1"

    # Autorisation d'accès au dossier physique correspondant à l'alias
    <Directory "/var/www/html/rep1">
        Require all granted
    </Directory>
```
*Note :* Pour `/rep2`, il n'y a rien à faire. Le chemin URL correspond "naturellement" au chemin physique `/var/www/html/web/rep2` car il est situé directement dans le `DocumentRoot` du site web.

Rechargez Apache : `systemctl reload apache2`.

#### Répertoires de publication des utilisateurs (UserDir)

**1. Activer le module `userdir` :**
```bash
a2enmod userdir
systemctl restart apache2
```

**2. Créer l'utilisateur `foo` et son espace web :**
```bash
# Création de l'utilisateur avec son répertoire personnel (-m)
useradd -m foo

# Connexion (facultative, ou on peut le faire en root) pour créer le répertoire web
mkdir /home/foo/public_html

# Création d'une page de test
echo "<h1>Bienvenue chez foo !</h1>" > /home/foo/public_html/index.html

# On s'assure des bonnes permissions (le serveur web 'www-data' doit pouvoir traverser le /home/foo)
chmod 755 /home/foo
chmod 755 /home/foo/public_html
chmod 644 /home/foo/public_html/index.html
```

**3. Test :**
Allez sur `http://wwwX.fai.com/~foo/` depuis `vm-browser` et la page "Bienvenue chez foo !" devrait s'afficher.