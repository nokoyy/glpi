# Installer / configurer Samba en tant que controlleur de domaine sur Debian 11
```
https://www.infotrucs.fr/tuto-configurer-samba-4-sous-debian-11/
```
Pour commencer faites un "ip addr" dans le terminal et regarder 
la configuration du port internet enp3s0 et garder son IP, car elle sera utile pour la suite.

->Dans une configuration simple, le serveur nommé ici DCSRV fournira les service Active Directory ainsi qu’un service DNS intégré à Samba. Il n’est nul besoin d’installer un serveur DNS tiers. 

->Le serveur DCSRV ne possède qu’une seule interface réseau.

->N’installez pas le paquet "resolvconf" et assurez-vous qu’il ne soit pas installé et en fonction.


-Je suis sur un réseau local tout à fait basique et la passerelle internet est ma box 
FAI sur l’IP locale 192.168.0.1. ( A CHANGER )

-Notez que pour l’instant, on va entrer l’IP d’un DNS public. 

-Le nom de domaine sera infotrucs.lan

-L’interface du serveur aura l’IP 192.168.0.100 (A CHANGER) avec un masque en 24 bits soit 255.255.255.0.

-Voici le contenu de mon fichier /etc/network/interfaces :
```
nano /etc/network/interfaces
```
source /etc/network/interfaces.d/*

#The loopback network interface

auto lo

iface lo inet loopback

allow-hotplug enp0s3

iface enp0s3 inet static

address 192.168.0.100

netmask 255.255.255.0

gateway 192.168.0.1

dns-nameservers 127.0.0.1


## Definir le hostname

Il faut définir un nom d’hôte pour le serveur si ce n’est déjà fait. Pour cela éditer le fichier :
```
nano /etc/hostname
```
-Entrez le nom sans le nom de domaine :

dcsrv

->Ne choisissez pas les noms PDC ni BDC.
Fichier hosts

-Éditez le fichier hosts pour la résolution de nom.
```
nano /etc/hosts
```
127.0.0.1 localhost localhost.localdomain
192.168.0.100 dcsrv.infotrucs.lan dcsrv

-Vous ne devez pas mettre la boucle locale 127.0.0.1 devant le vrai hostname du serveur, respectez bien l’exemple.

N’hésitez pas à redémarrer après la configuration du réseau.
Fichier resolv.conf

-Pour renseigner la zone de recherche DNS, éditez le fichier :
```
nano /etc/resolv.conf
```
-Et entrez :

domain infotrucs.lan
search infotrucs.lan
nameserver 127.0.0.1

## On passe en page suivante à l’installation de SAMBA.

```
apt-get update && apt-get upgrade
```
On passe à l’installation des dépendances.
```
apt-get install python perl acl xattr
```
```
apt-get install attr autoconf gdb bind9utils bison build-essential debhelper dnsutils docbook-xml docbook-xsl flex gdb libjansson-dev libacl1-dev libaio-dev libarchive-dev libattr1-dev libblkid-dev libbsd-dev libcap-dev libcups2-dev libgnutls28-dev libgpgme-dev libjson-perl libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl libpopt-dev libreadline-dev nettle-dev perl-modules-5.32 pkg-config python-all-dev python2-dbg python-dev-is-python2 python3-dnspython python3-gpg python3-markdown python3-dev xsltproc zlib1g-dev liblmdb-dev lmdb-utils docbook-xsl cups git libsasl2-dev libaio-dev libpam0g-dev valgrind autoconf ldap-utils krb5-user
```
-Des infos kerberos sont alors demandées (en majuscules !) (ici on n’a qu’un seul serveur) :

realm : INFOTRUCS.LAN

Serveurs kerberos du royaume : DCSRV.INFOTRUCS.LAN

Serveur administratif du royaume Kerberos : DCSRV.INFOTRUCS.LAN 

Nous sommes prêt pour l’installation. Toujours dans votre terminal en root :
```
apt-get install samba attr winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config libgssrpc4 libkadm5clnt-mit12 libkadm5srv-mit12 libkdb5-10 libldb2 libtalloc2 libtdb1 libtevent0 libwbclient0 python3-ldb python3-samba python3-talloc python3-tdb samba-common samba-common-bin samba-dsdb-modules samba-libs samba-vfs-modules tdb-tools krb5-doc ldb-tools smbldap-tools ufw
```
## -On va d’abord supprimer le smb.conf original :
rm /etc/samba/smb.conf

-Puis on lance la commande de promotion :
```
samba-tool domain provision –use-rfc2307 –server-role=dc –dns-backend=SAMBA_INTERNAL –realm=INFOTRUCS.LAN –domain=INFOTRUCS –adminpass=P@ssword01
```
->Le mot de passe défini dans cette commande est celui du compte AD administrator 
qui est admin du domaine. Il doit répondre aux exigences de complexité en contenant MAJUSCULES, 
minuscules, chiffres, et caractères spéciaux.

-La commande de promotion doit vous sortir quelque chose comme ça :

Server Role: active directory domain controller

Hostname: dcsrv

NetBIOS Domain: INFOTRUCS

DNS Domain: infotrucs.lan

DOMAIN SID: S-1-5-21-3461297074-3329795269-3091071446

## -Une fois l’opération terminée, on va remplacer le fichier krb5.conf d’origine par celui généré par samba-tool. D’abord, on supprime l’ancien :
```
rm /etc/krb5.conf
```
-Puis on copie celui généré par samba-tool. Son emplacement est affiché dans le résultat de la commande de promotion. Dans mon cas il se trouve ici :
```
cp /var/lib/samba/private/krb5.conf /etc/
```
-Bon, de toutes façons, on va grandement l’éditer. Adaptez en fonction de vos informations :
```
nano /etc/var/lib/samba/private/krb5.conf
```
[libdefaults]

default_realm = INFOTRUCS.LAN

dns_lookup_realm = true

dns_lookup_kdc = true

kdc_timesync = 1

ccache_type = 4

forwardable = true

proxiable = true

fcc-mit-ticketflags = true

[realms]

INFOTRUCS.LAN = {

kdc = dcsrv.infotrucs.lan

admin_server = dcsrv.infotrucs.lan

default_domain = infotrucs.lan

database_module = ldapconf

}

[domain_realm]

.infotrucs.lan = INFOTRUCS.LAN

infotrucs.lan = INFOTRUCS.LAN

->Enregistrez le fichier.

