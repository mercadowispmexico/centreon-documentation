---
id: secure-platform
title: Sécurisez votre plateforme
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


Ce chapitre vous propose de sécuriser votre plateforme Centreon.

## Renforcez la sécurité des comptes utilisateurs

Après l'installation de Centreon, il est nécessaire de changer les mots de passe par défaut des utilisateurs suivants:

- root
- centreon
- centreon-engine
- centreon-broker
- centreon-gorgone

Pour cela, utilisez la commande suivante avec un compte privilégié (par exemple sudo) ou avec root (non recommandé - vous devez
avoir un utilisateur dédié) :

```shell
passwd <account_name>
```

De plus, il est important de vérifier que le compte Apache ne dispose pas de droits de connexion au terminal. Exécutez
la commande suivante :

```shell
cat /etc/passwd | grep apache
```

Vous devez avoir **/sbin/nologin** tel que :

```shell
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

> Pour rappel, la liste des utilisateurs et des groupes se trouve [ici](../installation/prerequisites.md#utilisateurs-et-groupes)

## Activer SELinux

Centreon a récemment développé des règles SELinux afin de renforcer le contrôle
des composants par le système d'exploitation.

> Ces règles sont actuellement en **mode bêta** et peuvent être activées. Vous
> pouvez les activer en suivant cette procédure. Lors de la détection d'un
> problème, il est possible de désactiver SELinux globalement et de nous envoyer
> vos commentaires afin d'améliorer nos règles sur
> [Github](https://github.com/centreon/centreon).

### Présentation de SELinux

Security Enhanced Linux (SELinux) fournit une couche supplémentaire de sécurité du système. SELinux répond
fondamentalement à la question: `Le <suject> peut-il faire cette <action> sur <object> ?`, Par exemple: un serveur Web
peut-il accéder aux fichiers des répertoires personnels des utilisateurs ?

La stratégie d'accès standard basée sur l'utilisateur, le groupe et d'autres autorisations, connue sous le nom de
contrôle d'accès discrétionnaire (DAC), ne permet pas aux administrateurs système de créer des stratégies de sécurité
complètes et précises, telles que la restriction d'applications spécifiques à l'affichage uniquement des fichiers
journaux, tout en permettant à d'autres applications d'ajouter de nouvelles données aux fichiers journaux.

SELinux implémente le contrôle d'accès obligatoire (MAC). Chaque processus et ressource système possède une étiquette
de sécurité spéciale appelée contexte SELinux. Un contexte SELinux, parfois appelé étiquette SELinux, est un identifiant
qui fait abstraction des détails au niveau du système et se concentre sur les propriétés de sécurité de l'entité. Non
seulement cela fournit un moyen cohérent de référencer des objets dans la stratégie SELinux, mais cela supprime également
toute ambiguïté qui peut être trouvée dans d'autres méthodes d'identification. Par exemple, un fichier peut avoir plusieurs
noms de chemin valides sur un système qui utilise des montages de liaison.

La politique SELinux utilise ces contextes dans une série de règles qui définissent comment les processus peuvent
interagir entre eux et avec les différentes ressources système. Par défaut, la stratégie n'autorise aucune interaction
à moins qu'une règle n'accorde explicitement l'accès.

Pour plus d'informations à propos de SELinux, visitez la [documentation Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/getting-started-with-selinux_using-selinux)

### Activer SELinux en mode permissif

Par défaut, SELinux est désactivé lors du processus d'installation de Centreon. Pour activer SELinux en mode permissif,
vous devez modifier le fichier `/etc/selinux/config` comme tel :

```shell
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

Puis redémarrez votre serveur :
```shell
shutdown -r now
```

### Installer les paquets Centreon SELinux

Suivant le type de serveur, installer les paquets avec la commande suivante :

<Tabs groupId="sync">
<TabItem value="Central / Remote Server" label="Central / Remote Server">

   ```shell
   yum install centreon-common-selinux \
   centreon-web-selinux \
   centreon-broker-selinux \
   centreon-engine-selinux \
   centreon-gorgoned-selinux \
   centreon-plugins-selinux
   ```

</TabItem>
<TabItem value="Poller" label="Poller">

   ```shell
   yum install centreon-common-selinux \
   centreon-broker-selinux \
   centreon-engine-selinux \
   centreon-gorgoned-selinux \
   centreon-plugins-selinux
   ```

</TabItem>
<TabItem value="Map server" label="Map server">

   ```shell
   yum install centreon-map-selinux
   ```

</TabItem>
<TabItem value="MBI server" label="MBI server">

   ```shell
   yum install centreon-mbi-selinux
   ```

</TabItem>
</Tabs>

Pour vérifier l'installation, exécutez la commande suivante :

```shell
semodule -l | grep centreon
```

Suivant votre type de serveur, vous pouvez voir :
```shell
centreon-broker	0.0.5
centreon-common	0.0.10
centreon-engine	0.0.8
centreon-gorgoned	0.0.3
centreon-plugins	0.0.2
centreon-web	0.0.8
```

### Auditer les journaux et activer SELinux

Avant d'activer SELinux en **mode renforcé**, vous devez vous assurer qu'aucune erreur n'apparaît à l'aide de la
commande suivante :
```shell
cat /var/log/audit/audit.log | grep -i denied
```

Si des erreurs apparaissent, vous devez les analyser et décider si ces erreurs sont régulières et doivent être ajoutées
en plus des règles SELinux par défaut de Centreon. Pour ce faire, utilisez la commande suivante pour transformer
l'erreur en règles SELinux :

```shell
audit2allow -a
```

Exécutez ensuite les règles proposées.

Si après un certain temps, aucune erreur n'est présente, vous pouvez activer SELinux en mode renforcé en suivant cette
[procédure](#activer-selinux-en-mode-permissif) avec le mode **enforcing**.

> N'hésitez pas à nous faire part de vos retours sur [Github](https://github.com/centreon/centreon).

## Sécurisez les fichiers de configuration

Changez les permissions des fichiers de configuration suivants:

```
chown centreon:centreon /etc/centreon/conf.pm
chmod 660 /etc/centreon/conf.pm
```

et

```
chown apache:apache /etc/centreon/centreon.conf.php
chmod 660 /etc/centreon/centreon.conf.php
```

## Sécurisez l'installation du SGBD

[MariaDB](https://mariadb.com/kb/en/mysql_secure_installation/) propose une procédure par défaut pour sécuriser
l'installation du SGBD. Vous devez obligatoirement définir un mot de passe pour l'utilisateur **root** de la base de données. Veuillez exécuter la commande suivante et suivre les instructions :

```shell
mysql_secure_installation
```

## Activation de firewalld

Installez firewalld:
```shell
yum install firewalld
```

Activez firewalld:
```shell
systemctl enable firewalld
systemctl start firewalld
```

> La liste des flux réseau nécessaires pour chaque type de serveur est définie
> [ici](../installation/architectures.md#tableau-des-flux-de-la-plate-forme).

<Tabs groupId="sync">
<TabItem value="Central / Remote Server" label="Central / Remote Server">

Exemple de règles pour un Centreon Central ou Remote Server:
```shell
# For default protocols
firewall-cmd --zone=public --add-service=ssh --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=snmp --permanent
firewall-cmd --zone=public --add-service=snmptrap --permanent
# Centreon Gorgone
firewall-cmd --zone=public --add-port=5556/tcp --permanent
# Centreon Broker
firewall-cmd --zone=public --add-port=5669/tcp --permanent
```

</TabItem>
<TabItem value="Poller" label="Poller">

Exemple de règles pour un collecteur Centreon:
```shell
# For default protocols
firewall-cmd --zone=public --add-service=ssh --permanent
firewall-cmd --zone=public --add-service=snmp --permanent
firewall-cmd --zone=public --add-service=snmptrap --permanent
```

</TabItem>
</Tabs>

Une fois les règles ajoutées, il est nécessaire de recharger firewalld:
```shell
firewall-cmd --reload
```

### Activez fail2ban

Fail2ban est un framework de prévention contre les intrusions, écrit en Python.

Installez fail2ban :
```shell
yum install epel-release
yum install fail2ban fail2ban-systemd python-inotify
```

Si SELinux est installé, mettez à jour les politiques SELinux :
```shell
yum update -y selinux-policy*
```

Activez firewalld :
```shell
systemctl enable fail2ban
systemctl start fail2ban 
```

Copiez le fichier de règles par défaut :
```shell
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Editez le fichier `/etc/fail2ban/jail.local` et recherchez le bloc **[centreon]**, puis modifiez tel que :
```shell
[centreon]
port    = http,https
logpath = /var/log/centreon/login.log
backend  = pyinotify
```

Pour activer la règle **centreon** fail2ban, créez le fichier `/etc/fail2ban/jail.d/custom.conf` et ajoutez les lignes
suivantes :
```shell
[centreon]
enabled = true
findtime = 10m
bantime = 10m
maxretry = 3
```

> **maxretry** est le nombre d'authentifications échouées avant bannissement de l'adresse IP.
>
> **bantime** est la durée du bannissement.
>
> **findtime** est la plage de temps pour trouver les authentifications en échecs.

Puis redémarrez fail2ban pour charger votre règle :
```shell
systemctl restart fail2ban
```

Pour vérifier l'état de la règle **centreon**, vous pouvez exécuter :
```shell
fail2ban-client status centreon
Status for the jail: centreon
|- Filter
|  |- Currently failed:	1
|  |- Total failed:	17
|  `- File list:	/var/log/centreon/login.log
`- Actions
   |- Currently banned:	0
   |- Total banned:	2
   `- Banned IP list:
```

> Pour plus d'informations, visitez le [site officiel](http://www.fail2ban.org).

## Passez le serveur web en HTTPS

Par défaut, Centreon installe un serveur Web en mode HTTP. Il est fortement recommandé de passer en mode HTTPS en
ajoutant votre certificat.

Il vaut mieux utiliser un certificat validé par une autorité plutôt que d'utiliser un certificat auto-signé. Cependant, si un certificat auto-signé vous convient mieux, référez-vous à la [section correspondante](#sécurisez-le-serveur-web-apache-avec-un-certificat-auto-signé).

Si vous ne disposez pas d'un certificat validé par une autorité, vous pouvez en générer un sur des plateformes telles
que [Let's Encrypt](https://letsencrypt.org/).

> Une fois que votre serveur web est configuré en mode HTTPS et si vous avez un serveur Map sur votre plateforme, 
vous devez le configurer en mode HTTPS également, sinon, les navigateurs web récents peuvent bloquer 
la communication entre les deux serveurs. La procédure est détaillée [ici](../graph-views/secure-your-map-platform.md#Configure-HTTPS/TLS-on-the-MAP-server).

Une fois votre certificat obtenu, effectuez la procédure suivante pour activer le mode HTTPS sur votre serveur Apache :

1. Installez le module SSL pour Apache

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

```shell
dnf install mod_ssl mod_security openssl
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

```shell
yum install httpd24-mod_ssl httpd24-mod_security openssl
```

</TabItem>
</Tabs>

2. Installez vos certificats

Copiez votre certificat et votre clé sur le serveur en fonction de votre configuration; par défaut:

- /etc/pki/tls/certs/ca.crt
- /etc/pki/tls/private/ca.key

3. Sauvegardez la configuration actuelle du serveur Apache pour Centreon

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

```shell
cp /etc/httpd/conf.d/10-centreon.conf{,.origin}
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

```shell
cp /opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf{,.origin}
```

</TabItem>
</Tabs>

4. Éditez la configuration Apache pour Centreon

> Centreon propose un fichier de configuration d'exemple HTTPS disponible dans le répertoire:
> **/usr/share/centreon/examples/centreon.apache.https.conf**

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

Éditez  le fichier **/etc/httpd/conf.d/10-centreon.conf** tel que :

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

Éditez  le fichier **/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf** tel que :
</TabItem>
</Tabs>

```apacheconf
Alias /centreon/api /usr/share/centreon
Alias /centreon /usr/share/centreon/www/

<LocationMatch ^/centreon/(?!api/latest/|api/beta/|api/v[0-9]+/|api/v[0-9]+\.[0-9]+/)(.*\.php(/.*)?)$>
    ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/www/$1
</LocationMatch>

<LocationMatch ^/centreon/(authentication|api/(latest|beta|v[0-9]+|v[0-9]+\.[0-9]+))/.*$>
    ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/api/index.php/$1
</LocationMatch>

ProxyTimeout 300

<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
</VirtualHost>

<VirtualHost *:443>
#####################
# SSL configuration #
#####################
    SSLEngine On
    SSLProtocol All -SSLv3 -SSLv2 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-DSS-AES256-GCM-SHA384:DHE-DSS-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-GCM-SHA256:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!ADH:!IDEA
    SSLHonorCipherOrder On
    SSLCompression Off
    SSLCertificateFile /etc/pki/tls/certs/ca.crt
    SSLCertificateKeyFile /etc/pki/tls/private/ca.key

    <IfModule mod_security2.c>
        # https://github.com/SpiderLabs/ModSecurity/issues/652
        SecRuleRemoveById 200003
    </IfModule>

    <Directory "/usr/share/centreon/www">
        DirectoryIndex index.php
        Options Indexes
        AllowOverride all
        Order allow,deny
        Allow from all
        Require all granted
        <IfModule mod_php5.c>
            php_admin_value engine Off
        </IfModule>

        FallbackResource /centreon/index

        AddType text/plain hbs
    </Directory>

    <Directory "/usr/share/centreon/api">
        Options Indexes
        AllowOverride all
        Order allow,deny
        Allow from all
        Require all granted
        <IfModule mod_php5.c>
            php_admin_value engine Off
        </IfModule>

        AddType text/plain hbs
    </Directory>
</VirtualHost>

RedirectMatch ^/$ /centreon
```

> N'oubliez pas de changer les directives **SSLCertificateFile** et **SSLCertificateKeyFile** avec les chemins d'accès 
> vers votre clé et votre certificat.

5. Activez les flags HttpOnly et Secure et cachez la signature du serveur

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

Éditez le fichier **/etc/httpd/conf.d/10-centreon.conf** et ajouter la ligne suivante :

```apacheconf
Header always edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure;SameSite=Strict
ServerSignature Off
ServerTokens Prod
```

Éditez le fichier **/etc/php.d/50-centreon.ini*** et désactivez le paramètre `expose_php` :

```phpconf
expose_php = Off
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

Éditez le fichier **/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf** et ajouter la ligne suivante :

```apacheconf
Header set X-Frame-Options: "sameorigin"
Header always edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure;SameSite=Strict
ServerSignature Off
ServerTokens Prod
TraceEnable Off
```

Éditez le fichier **/etc/php.d/50-centreon.ini** et désactivez le paramètre `expose_php` :

```phpconf
expose_php = Off
```

</TabItem>
</Tabs>

6. Cacher le répertoire par défaut /icons

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

Éditez le fichier **/etc/httpd/conf.d/autoindex.conf** et commentez la ligne suivante :

```apacheconf
#Alias /icons/ "/usr/share/httpd/icons/"
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

Éditez le fichier **/opt/rh/httpd24/root/etc/httpd/conf.d/autoindex.conf** et commentez la ligne suivante :

```apacheconf
#Alias /icons/ "/opt/rh/httpd24/root/usr/share/httpd/icons/"
```

</TabItem>
</Tabs>

7. Redémarrez le serveur web Apache et PHP pour prendre en compte la configuration

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

```shell
systemctl restart php-fpm httpd
```

Puis vérifiez le statut :

```shell
systemctl status httpd
```

Si tout est correct, vous devriez avoir quelque chose comme :

```shell
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Tue 2020-10-27 12:49:42 GMT; 2h 35min ago
     Docs: man:httpd.service(8)
 Main PID: 1483 (httpd)
   Status: "Total requests: 446; Idle/Busy workers 100/0;Requests/sec: 0.0479; Bytes served/sec: 443 B/sec"
    Tasks: 278 (limit: 5032)
   Memory: 39.6M
   CGroup: /system.slice/httpd.service
           ├─1483 /usr/sbin/httpd -DFOREGROUND
           ├─1484 /usr/sbin/httpd -DFOREGROUND
           ├─1485 /usr/sbin/httpd -DFOREGROUND
           ├─1486 /usr/sbin/httpd -DFOREGROUND
           ├─1487 /usr/sbin/httpd -DFOREGROUND
           └─1887 /usr/sbin/httpd -DFOREGROUND
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

```shell
systemctl restart php-fpm httpd24-httpd
```

Puis vérifiez le statut :

```shell
systemctl status httpd24-httpd
```

Si tout est correct, vous devriez avoir quelque chose comme :

```shell
● httpd24-httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd24-httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since mar. 2020-05-12 15:39:58 CEST; 25min ago
  Process: 31762 ExecStop=/opt/rh/httpd24/root/usr/sbin/httpd-scl-wrapper $OPTIONS -k graceful-stop (code=exited, status=0/SUCCESS)
 Main PID: 31786 (httpd)
   Status: "Total requests: 850; Idle/Busy workers 50/50;Requests/sec: 0.547; Bytes served/sec: 5.1KB/sec"
   CGroup: /system.slice/httpd24-httpd.service
           ├─ 1219 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31786 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31788 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31789 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31790 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31802 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31865 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31866 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31882 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─31903 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           └─32050 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
```

</TabItem>
</Tabs>

### Sécurisez le serveur web Apache avec un certificat auto-signé

Soit un serveur Centreon avec le FQDN suivant : **centreon7.localdomain**.

1. Préparer la configuration openssl :

    En raison d'un changement de politique chez Google, les certificats auto-signés peuvent être rejetés par le navigateur Google Chrome (sans qu'il soit possible d'ajouter une exception). Pour continuer à utiliser ce navigateur, vous devez modifier la configuration openssl.

    Ouvrez le fichier **/etc/pki/tls/openssl.cnf** et trouvez la section **[v3_ca]**:

    ```text
    # Ajoutez le tag alt_names qui vous permet d'indiquer les différents IPs et FQDNs du serveur
    [ alt_names ]
    IP.1 = xxx.xxx.xxx.xxx
    DNS.1 = centreon7.localdomain
    # Si vous avez plusieurs IPs(HA: vip + ip)
    # IP.2 = xxx.xxx.xxx.xxx
    [ v3_ca ]
    subjectAltName = @alt_names
    ```

2. Créer une clé privée pour le serveur :

    Créez une clé privée nommée **centreon7.key** sans mot de passe afin qu'elle puisse être utilisée par le service apache.

    ```text
    openssl genrsa -out centreon7.key 2048
    ```

    Protégez le fichier en modifiant ses droits :

    ```text
    chmod 400 centreon7.key
    ```

3. Créer un fichier CSR : 

    Avec la clé que vous venez de créer, créez un fichier CSR (Certificate Signing Request). Remplissez les champs avec les informations propres à votre entreprise.
    Le champ **Common Name** doit être identique au hostname de votre serveur Apache (dans notre cas, **centreon7.localdomain**).

    ```text
    openssl req -new -key centreon7.key -out centreon7.csr
    ```

4. Créer une clé privée pour le certificat de l'autorité de certification :

    En premier lieu, créez une clé privée pour cette autorité. Ajoutez l'option -aes256 pour chiffrer la clé produite et y appliquer un mot de passe. Ce mot de passe sera demandé chaque fois que la clé sera utilisée.

    ```text
    openssl genrsa -aes256 2048 > ca_demo.key
    ```

5. Créer le certificat x509 à partir de la clé privée du certificat de l'autorité de certification :

    Ensuite, créez un certificat x509 qui sera valide pendant un an.

    >  Attention, il est nécessaire de simuler un tiers de confiance : le "Common Name" doit être différent de celui du certificat du serveur.

    ```text
    openssl req -new -x509 -days 365 -key ca_demo.key -out ca_demo.crt
    ```

    Ce certificat étant créé, vous pourrez l'utiliser pour signer le certificat du serveur.

6. Créer un certificat pour le serveur :

    Utilisez le certificat x509 pour signer votre certificat pour le serveur :

    ```text
    openssl x509 -req -in centreon7.csr -out centreon7.crt -CA ca_demo.crt -CAkey ca_demo.key -CAcreateserial -CAserial ca_demo.srl  -extfile /etc/pki/tls/openssl.cnf -extensions v3_ca
    ```

    L'option CAcreateserial n'est nécessaire que la première fois. Vous devez entrer le mot de passe précédemment défini. Vous obtenez un certificat pour le serveur nommé **centreon7.crt**.

    Vous pouvez voir le contenu du fichier : 

    ```text
    less centreon7.crt
    ```

7. Copier les fichiers dans la configuration Apache :

    Copiez la clé privée du serveur et le certificat du serveur que vous avez signé.

    ```text
    cp centreon7.key /etc/pki/tls/private/centreon7.key
    cp centreon7.crt /etc/pki/tls/certs/
    ```

8. Mettre à jour le fichier de configuration Apache :

    Selon le nom des fichiers créés, mettez à jour les paramètres **SSLCertificateFile** et **SSLCertificateKeyFile** dans votre fichier de configuration Apache (**/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf**).

    Voici un exemple de ce à quoi le fichier peut ressembler:

    ```apacheconf
    Alias /centreon/api /usr/share/centreon
    Alias /centreon /usr/share/centreon/www/

    <LocationMatch ^/centreon/(?!api/latest/|api/beta/|api/v[0-9]+/|api/v[0-9]+\.[0-9]+/)(.*\.php(/.*)?)$>
        ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/www/$1
    </LocationMatch>

    <LocationMatch ^/centreon/(authentication|api/(latest|beta|v[0-9]+|v[0-9]+\.[0-9]+))/.*$>
        ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/api/index.php/$1
    </LocationMatch>

    ProxyTimeout 300

    <VirtualHost *:80>
        RewriteEngine On
        RewriteCond %{HTTPS} off
        RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
    </VirtualHost>

    <VirtualHost *:443>
        #####################
        # SSL configuration #
        #####################
        SSLEngine on
        SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
        SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
        SSLCertificateFile /etc/pki/tls/certs/centreon7.crt
        SSLCertificateKeyFile /etc/pki/tls/private/centreon7.key

        <IfModule mod_security2.c>
            # https://github.com/SpiderLabs/ModSecurity/issues/652
            SecRuleRemoveById 200003
        </IfModule>

        <Directory "/usr/share/centreon/www">
            DirectoryIndex index.php
            Options Indexes
            AllowOverride all
            Order allow,deny
            Allow from all
            Require all granted
            <IfModule mod_php5.c>
                php_admin_value engine Off
            </IfModule>

            FallbackResource /centreon/index

            AddType text/plain hbs
        </Directory>

        <Directory "/usr/share/centreon/api">
            Options Indexes
            AllowOverride all
            Order allow,deny
            Allow from all
            Require all granted
            <IfModule mod_php5.c>
                php_admin_value engine Off
            </IfModule>

            AddType text/plain hbs
        </Directory>
    </VirtualHost>
    ```
9. Copier le certificat x509 sur le navigateur client :

    Maintenant, récupérez le fichier du certificat x509 **ca_demo.crt** et importez-le dans le magasin de certificats de votre navigateur.

## URI personnalisée

Il est possible de modifier l'URI de Centreon. Par exemple, **/centreon** peut être remplacé par **/monitoring**.

> Au moins un niveau de chemin est obligatoire.

Pour mettre à jour l'URI Centreon, vous devez suivre les étapes suivantes:

1. Rendez-vous dans le menu `Administration > Paramètres > Centreon web` et modifiez le champ **Centreon Web Directory**

![image](../assets/administration/custom-uri.png)

2. Éditez le fichier de configuration Apache pour Centreon

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

```shell
vim /etc/httpd/conf.d/10-centreon.conf
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

```shell
vim /opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf
```

</TabItem>
</Tabs>

et modifiez le chemin **/centreon** par le nouveau.

## Activation du http2

Il est possible d'activer le protocole http2 pour améliorer les performances réseaux de Centreon.

Pour utiliser http2, vous devez suivre les étapes suivantes:

<Tabs groupId="sync">
<TabItem value="RHEL / CentOS / Oracle Linux 8" label="RHEL / CentOS / Oracle Linux 8">

1. [Configurer le https pour Centreon](./secure-platform.md#passer-le-serveur-web-en-https)

2. Installer le module nghttp2:

```shell
dnf install nghttp2
```

3. Enable http2 protocol in **/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf**:

```apacheconf
...
<VirtualHost *:443>
    Protocols h2 h2c http/1.1
    ...
</VirtualHost>
...
```

4. Modifier la méthode utilisée par apache pour le module multi-processus dans **/etc/httpd/conf.modules.d/00-mpm.conf** :

```diff
-LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
+#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

-#LoadModule mpm_event_module modules/mod_mpm_event.so
+LoadModule mpm_event_module modules/mod_mpm_event.so
```

5. Redémarrer le processus Apache pour prendre en compte la nouvelle configuration:

```shell
systemctl restart httpd
```

</TabItem>
<TabItem value="CentOS 7" label="CentOS 7">

1. [Configurer le https pour Centreon](./secure-platform.md#passer-le-serveur-web-en-https)

2. Installer le module nghttp2:

```shell
yum install httpd24-nghttp2
```

3. Enable http2 protocol in **/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf**:

```apacheconf
...
<VirtualHost *:443>
    Protocols h2 h2c http/1.1
    ...
</VirtualHost>
...
```

4. Modifier la méthode utilisée par apache pour le module multi-processus dans **/opt/rh/httpd24/root/etc/httpd/conf.modules.d/00-mpm.conf**:

```diff
-LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
+#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

-#LoadModule mpm_event_module modules/mod_mpm_event.so
+LoadModule mpm_event_module modules/mod_mpm_event.so
```

5. Redémarrer le processus Apache pour prendre en compte la nouvelle configuration:

```shell
systemctl restart httpd24-httpd
```

</TabItem>
</Tabs>

## Authentification des utilisateurs

Centreon propose plusieurs méthodes pour authentifier les utilisateurs :

- [localement](../connect/loginpwd.md) (MySQL)
- [LDAP](./parameters/ldap.md)
- [Generic SSO](../connect/sso.md) ou [OpenId Connect](../connect/openid.md)

## Créer des profils d'utilisateurs

Centreon propose de gérer les autorisations d'accès aux différents menus, ressources et actions possibles sur ces ressources
via la gestion de [liste de contrôle d'accès](./access-control-lists.md).

## Communications sécurisées entre les serveurs

Il est fortement recommandé de sécuriser les communications entre les différents serveurs de la plateforme Centreon si
certains serveurs ne sont pas dans un réseau sécurisé.

> Le tableau des flux réseau est disponible [ici](../installation/architectures.md#tableau-des-flux-réseau).

### Communication Centreon Broker 

#### Centreon Broker et pare-feu

Parfois, il n'est pas possible d'initialiser le flux Centreon Broker depuis le collecteur (ou Remote Server)
vers le serveur Centreon Central ou le Remote Server.
[Voir la configuration suivante pour inverser le flux](../monitoring/monitoring-servers/advanced-configuration.md#centreon-broker-and-the-firewall).

#### Authentification des flux Centreon Broker

Si vous devez authentifier des collecteurs qui envoient des données, vous pouvez utiliser le mécanisme d'authentification
Centreon Broker, qui est basé sur des certificats X.509.
[Voir la configuration suivante pour authentifier les collecteurs](../monitoring/monitoring-servers/advanced-configuration.md#authentification-avec-centreon-broker).

#### Compressez et chiffrez la communication Centreon Broker

Il est également possible de compresser et de chiffrer la communication de Centreon Broker. Allez dans le menu
`Configuration > Pollers > Broker configuration`, modifiez votre configuration Centreon Broker et activez les entrées
et sorties **IPv4**:

- Enable TLS encryption: Auto
- Enable negotiation: Yes
- Compression (zlib): Auto

### Communication Centreon Gorgone

La [documentation officielle de Centreon gorgone](https://github.com/centreon/centreon-gorgone/blob/master/docs/configuration.md#gorgonecore) vous permettra de sécuriser la communication entre les processus Gorgone.

## Gestion de l'information et des événements de sécurité (SIEM)

Les journaux des événements Centreon sont disponibles dans les répertoires suivants :

| Répertoires des journaux  | Central server | Remote Server | Poller | Centreon Map server | Centreon MBI Server |
|---------------------------|----------------|---------------|--------|---------------------|---------------------|
| /var/log/centreon         | X              | X             |        |                     |                     |
| /var/log/centreon-broker  | X              | X             | X      |                     |                     |
| /var/log/centreon-engine  | X              | X             | X      |                     |                     |
| /var/log/centreon-gorgone | X              | X             | X      |                     |                     |
| /var/log/centreon-bi      | X              | X             |        |                     |                     |
| /var/log/centreon-map     | X              | X             |        | X                   | X                   |

> De plus, toutes les actions de modification de la configuration de Centreon effectuées par les utilisateurs sont
> disponibles via le menu [`Administration > Logs`](./logging-configuration-changes.md).

# Sauvegardez votre plateforme

Centreon propose de sauvegarder la configuration de la plateforme. Pour ce faire, accédez au menu
[`Administration  >  Parameters  >  Backup`](./backup.md).
