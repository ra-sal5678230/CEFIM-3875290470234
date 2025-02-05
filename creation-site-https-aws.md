SALGUEIRO raphael, procédure expliquant la mise en place des prérequis pour le cefim


LE LIEN FINAL DU SITE EST "https://site-cefim-trop-sympa.online/"

# **AWS :**

voir image_1
# **mise en place du site web :**
installation des premières dépendances pour la mise en place initiale d'un site en http 

```
apt update && apt upgrade 
apt install php-myadmin
apt install apache2 php mariadb-server 
apt install php-intl
```

avec wget je vais choisir un glpi

```
wget https://github.com/glpi-project/glpi/releases/download/10.0.17/glpi-10.0.17.tgz
```

décompression :
`tar -xvf glpi-10.0.17.tgz glpi/`

création du vhost http :
`sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/glpi.conf`
`sudo nano /etc/apache2/sites-available/glpi.conf`
**modification :** 

```
<VirtualHost *:80>
    ServerName localhost

    DocumentRoot /var/www/glpi/public

    # If you want to place GLPI in a subfolder of your site (e.g. your virtual host is serving multiple applications),
    # you can use an Alias directive. If you do this, the DocumentRoot directive MUST NOT target the GLPI directory itself.
    # Alias "/glpi" "/var/www/glpi/public"

    <Directory /var/www/glpi/public>
        Require all granted

        RewriteEngine On

        # Ensure authorization headers are passed to PHP.
        # Some Apache configurations may filter them and break usage of API, CalDAV, ...
        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

        # Redirect all requests to GLPI router, unless file exists.
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>
```

puis on active le VHOST avec les commande suivante :

`a2ensite glpi.conf`
`a2enmod rewrite`
`systemctl restart apache2`


copie du glpi dans le bon endroit :
`cp -r /la_position_du_glpi/gpli /var/www/glpi`

application des droits a glpi pour éviter les gros problèmes avec "chmod"

# **mise en place HTTPS**

en premier temps pour certbot on va prendre un nom de domaine chez aws :
voir image_2

on change la zone dns pour mettre la bonne adresse ip du serveur web pour une bonne redirection 

voir image_3

après quelques temps le nom de domaine fonctionne bien (il faut changer celui du glpi.conf pour qu'il soit le même que le nom de domaine acheté)

voir image_4

#### **installation de snap et certbot :** 
```
sudo apt install snapd
sudo snap install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
sudo certbot renew --dry-run
```

Dans la commande certbot --apache, il y a un email à fournir et surtout le domaine acheté à OVH.
La sécurisation du transit se fera donc ensuite avec une mise en place des certificats, le site est donc sécurisé, pour finir, il ne faudra qu'ouvrir glpi.conf et y faire une redirection en https pour forcer son utilisation
voir image_5

pour finir, l'installation de webmin qui est quelque peut problèmatique étant donné qu'il faut rajouter nos propres sources.list :

wget -qO - http://www.webmin.com/jcameron-key.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://download.webmin.com/download/repository sarge contrib" > \
/etc/apt/sources.list.d/webmin.list

Maitenant, on peut l'installer avec :

sudo apt install webmin

et on l'active avec :
sudo systemctl start webmin
sudo systemctl enable webmin




tout l'historique de toutes les commandes passées est dans mon fichier "historique"
