# GLPI - Debian 11


 Après l'installation de Debian sur la machine on cherhce à créer un nouvel utilisateur, pour se faire on se rend dans "Paramètres" -> "Utilisateurs" puis
"Ajouter un nouvel utilisateur" auquel on donnera la permission Administrateur. Puis on change d'utilisateur pour se connecter avec ce dernier.

On va commencer à taper les lignes de code en root donc on tape la commande :
```
sudo -i
```
On met à jour la machine :
```
apt update && apt upgrade -y
```

On va installer les applications nécessaires, à savoir apache2 pour les services web, mariadb pour la base de données et php pour le langage de programmation (la machine devient donc un serveur « LAMP »).

```
apt install apache2 php libapache2-mod-php mariadb-server -y
```
Ensuite, nous allons installer toutes les dépendances donc pourrait avoir besoin GLPI (elles ne sont pas toutes obligatoires/utiles mais pour éviter les problèmes par la suite, nous installons tout d’un coup ).

```
apt install php-mysqli php-mbstring php-curl php-gd php-simplexml php-intl php-ldap php-apcu php-xmlrpc php-cas php-zip php-bz2 php-imap -y
```
Voilà qui est fait. Nous allons maintenant sécuriser l’accès au service de base de données. Lancez la commande suivante :
```
mysql_secure_installation
```
On appuie sur "entrée" pour la première requête puis "y", on entre notre mdp et pour tout le reste on appuye sur "y"

Maintenant que l’accès aux bases de données est sécurisé, nous allons pouvoir nous y connecter avec le compte root et le mot de passe que nous venons de lui attribuer :
```
mysql -u root -p
```

Il faut créer la base de données qui sera utilisée par GLPI et un utilisateur de base de données qui aura les pleins pouvoirs sur celle-ci. Voici les 3 commandes à saisir pour cela (les ; sont nécessaires) :
```
create database bdd_glpi;
grant all privileges on bdd_glpi.* to adminbdd_glpi@localhost identified by "votre-MDP";
exit
```
Quelques explications rapides sur ces commandes :

La 1ère va créer une base de données appelée «  bdd_glpi », à vous de donner le nom qu’il vous plaira.
  
  La 2nde va à la fois créer un utilisateur ici nommé « adminbdd_glpi », lui attribuer le mot de passe « votre-MDP » et lui donner tous les privilèges (une sorte de « contrôle total » sur la base de données « bdd_glpi »). Une fois encore, à vous de définir les noms que vous souhaitez.
 
 La commande exit (ou quit) sert simplement à quitter le service SQL et revenir dans le terminal.

Passons maintenant à l’installation de GLPI !

Placez vous dans le répertoire de votre choix (moi perso c’est dans le dossier temporaire /tmp) et téléchargez la dernière version disponible de GLPI sur Github :
```
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/10.0.7/glpi-10.0.7.tgz
```

Décompressez l’archive de GLPI :
```
tar -xvzf glpi-10.0.7.tgz
```
Copiez le contenu du dossier décompressé nommé « glpi » dans /var/www/html (vous pouvez aussi le déplacer directement mais j’aime bien conserver temporairement une copie propre de ce que j’installe sous Linux… vieille habitude ^^) et supprimez au passage le fichier index.html qui n’est autre qu’une sorte de page d’accueil d’apache :
```
rm /var/www/html/index.html
cp -r glpi/* /var/www/html/
```
Rendez l’utilisateur des services web (nommé www-data) propriétaire de ces nouveaux fichiers :
```
chown -R www-data /var/www/html
```
Il ne reste plus qu’à redémarrer le service apache2 pour appliquer toutes les modifications apportées :
```
service apache2 restart

```
Pour finir on peut aller sur notre naviguateur et taper l'addresse ip de notre machine pour suivi de /glpi pour se connecter directement au serveur.
La commande pour connaître son addresse ip est la suivante :


```
ip addr
```


Exemple : 192.168.1.1/glpi









