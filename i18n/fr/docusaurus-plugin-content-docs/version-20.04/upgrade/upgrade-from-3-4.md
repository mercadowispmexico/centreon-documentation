---
id: upgrade-from-3-4
title: Montée de version depuis Centreon 3.4
---

Ce chapitre décrit la procédure de montée de version de votre plate-forme
Centreon depuis la version 3.4 (Centreon Web 2.8) vers la version 20.04.

> A la fin de cette procédure, les utilisateurs de Centreon EMS devront demander
> de nouvelles licenses au [Support Centreon](https://centreon.force.com).

> Cette procédure ne s'applique que pour une plate-forme Centreon installée à
> partir des dépôts Centreon 3.4 sur des distributions **Red Hat / CentOS en
> version 7**.
>
> Si cela n'est pas le cas, se référer à la
> [procédure de migration](../migrate/migrate-from-3-4).

## Sauvegarde

Avant toute chose, il est préférable de s’assurer de l’état et de la consistance
des sauvegardes de l’ensemble des serveurs centraux de votre plate-forme :

- Serveur Centreon Central,
- Serveur de gestion de base de données.

## Mettre à jour la clé de signature RPM

Pour des raisons de sécurité, les clés utilisées pour signer les RPMs Centreon sont changées régulièrement. Le dernier changement a eu lieu le 14 octobre 2021. Lorsque vous mettez Centreon à jour depuis une version plus ancienne, vous devez suivre la [procédure de changement de clé](../security/key-rotation#installation-existante), afin de supprimer l'ancienne clé et d'installer la nouvelle.

## Montée de version du serveur Centreon central

### Installation des dépôts

#### Dépôt *Software collections* de Red Hat

Afin d'installer les logiciels Centreon, le dépôt *Software collections* de Red
Hat doit être activé.

> Le dépôt *Software collections* est nécessaire pour l'installation de PHP 7
> et les librairies associées.

Exécutez la commande suivante :

``` shell
yum install -y centos-release-scl
```

### Mise à jour des dépôts

Il est nécessaire de mettre à jour le dépôt Centreon.

Exécutez la commande suivante :

```shell
yum install -y https://yum.centreon.com/standard/20.04/el7/stable/noarch/RPMS/centreon-release-20.04-1.el7.centos.noarch.rpm
```

### Montée de version de la solution Centreon

> Centreon 20.04 utilise **MariaDB 10.3**.
>
> Le processus suivant met seulement à jour les composants Centreon pour le
> moment.
>
> MariaDB sera mis à jour après.

Mettez à jour le cache de yum :

```shell
yum clean all --enablerepo=*
```

Mettez à jour l'ensemble des composants :

```shell
yum update centreon\*
```

> Acceptez les nouvelles clés GPG des dépôts si nécessaire.

### Actions complémentaires

#### Mise à jour de la version de PHP

Centreon 20.04 utilise un nouveau paquet PHP.

Le fuseau horaire par défaut de PHP 7 doit être configuré. Executez la commande
suivante :

```shell
echo "date.timezone = Europe/Paris" >> /etc/opt/rh/rh-php72/php.d/50-centreon.ini
```

> Changez **Europe/Paris** par votre fuseau horaire. La liste des fuseaux
> horaires est disponible [ici](http://php.net/manual/en/timezones.php).

Réalisez les actions suivantes :

```shell
systemctl enable rh-php72-php-fpm
systemctl start rh-php72-php-fpm
```

#### Mise à jour du serveur Web Apache

Centreon 20.04 utilise un nouveau paquet pour le serveur Web Apache.

> Si vous avez modifié la configuration, reportez celle-ci dans le répertoire
> **/opt/rh/httpd24/root/etc/httpd/conf.d/**.

Si SSL était activé, installer le paquet suivant :

```shell
yum install httpd24-mod_ssl
```

Puis réalisez les actions suivantes :

```shell
systemctl disable httpd
systemctl stop httpd
systemctl enable httpd24-httpd
systemctl start httpd24-httpd
```

#### Configurer l'accès à l'API

Si vous aviez une configuration personnalisée, le processus de mise à jour RPM
n'y a pas touché.

> Si vous utilisez le https, vous pouvez suivre
> [cette procédure](../administration/secure-platform#sécurisez-le-serveur-web-apache)

Vous devez donc ajouter la section d'accès à l'API dans votre fichier de
configuration apache : **/opt/rh/httpd24/root/etc/httpd/conf.d/10-centreon.conf**

Seules les lignes avec le symbole "+" doivent être prises en compte.

```diff
+Alias /centreon/api /usr/share/centreon
Alias /centreon /usr/share/centreon/www/

+<LocationMatch ^/centreon/(?!api/latest/|api/beta/|api/v[0-9]+/|api/v[0-9]+\.[0-9]+/)(.*\.php(/.*)?)$>
+  ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/www/$1
+</LocationMatch>

+<LocationMatch ^/centreon/api/(latest/|beta/|v[0-9]+/|v[0-9]+\.[0-9]+/)(.*)$>
+  ProxyPassMatch fcgi://127.0.0.1:9042/usr/share/centreon/api/index.php/$1
+</LocationMatch>

ProxyTimeout 300

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

+    FallbackResource /centreon/index

    AddType text/plain hbs
</Directory>

+<Directory "/usr/share/centreon/api">
+    Options Indexes
+    AllowOverride all
+    Order allow,deny
+    Allow from all
+    Require all granted
+    <IfModule mod_php5.c>
+        php_admin_value engine Off
+    </IfModule>
+
+    AddType text/plain hbs
+</Directory>

RedirectMatch ^/$ /centreon
```

Redémarrez ensuite le service Apache :

```shell
systemctl restart httpd24-httpd
```

### Synchronisation des plugins

> La macro de ressource $USER1$ de Centreon 20.04 pointe à présent sur /usr/lib64/nagios/plugins.

Afin de résoudre cette situation, lancez les commandes suivantes:

```shell
mv /usr/lib64/nagios/plugins/* /usr/lib/nagios/plugins/
rmdir /usr/lib64/nagios/plugins/
ln -s -t /usr/lib64/nagios/ /usr/lib/nagios/plugins/
```

De cette façon un lien symbolique est créé :

```shell
$ ls -alt /usr/lib64/nagios/
lrwxrwxrwx   1 root root      24  1 nov.  17:59 plugins -> /usr/lib/nagios/plugins/
-rwxr-xr-x   1 root root 1711288  6 avril  2018 cbmod.so
```

### Finalisation de la mise à jour

Avant de démarrer la montée de version via l'interface web, rechargez le
serveur Apache avec la commande suivante :

```shell
systemctl reload httpd24-httpd
```

Connectez-vous ensuite à l'interface web Centreon pour démarrer le processus de
mise à jour :

Cliquez sur **Next** :

![image](../assets/upgrade/web_update_1.png)

Cliquez sur **Next** :

![image](../assets/upgrade/web_update_2.png)

La note de version présente les principaux changements, cliquez sur **Next** :

![image](../assets/upgrade/web_update_3.png)

Le processus réalise les différentes mises à jour, cliquez sur **Next** :

![image](../assets/upgrade/web_update_4.png)

Votre serveur Centreon est maintenant à jour, cliquez sur **Finish** pour
accéder à la page de connexion :

![image](../assets/upgrade/web_update_5.png)

Si le module Centreon BAM est installé, référez-vous à la [documentation
associée](../service-mapping/upgrade) pour le mettre à jour.

### Actions post montée de version

#### Montée de version des extensions

Depuis le menu `Administration > Extensions > Gestionnaire`, mettez à jour
toutes les extensions, en commençant par les suivantes :

  - License Manager,
  - Plugin Packs Manager,
  - Auto Discovery.

Vous pouvez alors mettre à jour toutes les autres extensions commerciales.

#### Démarrer le gestionnaire de tâches

Centreon 20.04 a changé son gestionnaire de tâches en passant de *Centcore* à
*Gorgone*.

Pour acter ce changement, réalisez les actions suivantes :

```shell
systemctl stop centcore
systemctl enable gorgoned
systemctl start gorgoned
```

Les statistiques Engine qui étaient collectées par *Centcore* le seront
maintenant par *Gorgone*

Il faut alors changer les droits sur les fichiers RRD de statistique en
exécutant la commande suivante:

```shell
chown -R centreon-gorgone /var/lib/centreon/nagios-perf/*
```

#### Redémarrage des processus de supervision

Le composant Centreon Broker a changé le format de son fichier de configuration.

Il utilise maintenant JSON à la place de XML.

Pour être sur que Broker et que le module Broker de Engine utilisent les nouveaux
fichiers de configuration, suivez ces étapes :

1. Déployer la configuration du Central depuis l'interface web en suivant
[cette procedure](../monitoring/monitoring-servers/deploying-a-configuration),
2. Redémarrer Broker et Engine sur le serveur Central en exécutant la commande
suivante:

  ```shell
  systemctl restart cbd centengine
  ```

### Montée de version du serveur MariaDB

Les composants MariaDB peuvent maintenant être mis à jour.

Sachez que MariaDB recommande vivement de monter en version le serveur en
passant par chacune des versions majeures. Veuillez vous référer à la [documentation officielle de MariaDB](https://mariadb.com/kb/en/upgrading/) pour plus d'informations.

Vous devez donc mettre à jour de la version 10.1 vers 10.2 puis 10.2 vers
10.3.

Pour cela, Centreon met à disposition les versions 10.2 et 10.3 sur ses
dépôts stables.

> Référez vous à la documentation officielle de MariaDB pour en savoir d'avantage
> sur ce processus :
>
> - https://mariadb.com/kb/en/upgrading-from-mariadb-101-to-mariadb-102/#how-to-upgrade
> - https://mariadb.com/kb/en/upgrading-from-mariadb-102-to-mariadb-103/#how-to-upgrade

#### Configuration

Le paramètre `innodb_additional_mem_pool_size` a été supprimé depuis MariaDB 10.2, vous devez donc le supprimer
du fichier **/etc/my.cnf.d/centreon.cnf**

```diff
#
# Custom MySQL/MariaDB server configuration for Centreon
#
[server]
innodb_file_per_table=1

open_files_limit = 32000

key_buffer_size = 256M
sort_buffer_size = 32M
join_buffer_size = 4M
thread_cache_size = 64
read_buffer_size = 512K
read_rnd_buffer_size = 256K
max_allowed_packet = 8M

# For 4 Go Ram
-#innodb_additional_mem_pool_size=512M
#innodb_buffer_pool_size=512M

# For 8 Go Ram
-#innodb_additional_mem_pool_size=1G
#innodb_buffer_pool_size=1G
```

#### Montée de version de 10.1 à 10.2

Suivez ces étapes résumées pour réaliser la montée de version comme MariaDB le
recommande :

1. Arrêtez le service mariadb :

    ```shell
    systemctl stop mariadb
    ```

2. Désinstallez la version actuelle 10.1 :

    ```shell
    rpm --erase --nodeps --verbose MariaDB-server MariaDB-client MariaDB-shared MariaDB-compat MariaDB-common
    ```

3. Installez la version 10.2 :

    ```shell
    yum install MariaDB-server-10.2\* MariaDB-client-10.2\* MariaDB-shared-10.2\* MariaDB-compat-10.2\* MariaDB-common-10.2\*
    ```

4. Démarrer le service mariadb :

    ```shell
    systemctl start mariadb
    ```

5. Lancez le processus de mise à jour MariaDB :

    ```shell
    mysql_upgrade
    ```

> Référez vous à la [documentation officielle](https://mariadb.com/kb/en/mysql_upgrade/)
> si des erreurs apparaissent pendant cette dernière étape.

#### Montée de version de 10.2 à 10.3

Suivez ces étapes résumées pour réaliser la montée de version comme MariaDB le
recommande :

1. Arrêtez le service mariadb :

    ```shell
    systemctl stop mariadb
    ```

2. Désinstallez la version actuelle 10.2 :

    ```shell
    rpm --erase --nodeps --verbose MariaDB-server MariaDB-client MariaDB-shared MariaDB-compat MariaDB-common
    ```

3. Installez la version 10.3 :

    ```shell
    yum install MariaDB-server-10.3\* MariaDB-client-10.3\* MariaDB-shared-10.3\* MariaDB-compat-10.3\* MariaDB-common-10.3\*
    ```

4. Démarrer le service mariadb :

    ```shell
    systemctl start mariadb
    ```

5. Lancez le processus de mise à jour MariaDB :

    ```shell
    mysql_upgrade
    ```

> Référez vous à la [documentation officielle](https://mariadb.com/kb/en/mysql_upgrade/)
> si des erreurs apparaissent pendant cette dernière étape.

#### Activer MariaDB au démarrage automatique

Exécutez la commande suivante :
```shell
systemctl enable mariadb
```

## Montée de version des Pollers

### Mise à jour des dépôts

Exécutez la commande suivante :

```shell
yum install -y https://yum.centreon.com/standard/20.04/el7/stable/noarch/RPMS/centreon-release-20.04-1.el7.centos.noarch.rpm
```

### Montée de version de la solution Centreon

Videz le cache de yum :

```shell
yum clean all --enablerepo=*
```

Mettez à jour l'ensemble des composants :

```shell
yum update centreon\*
```

> Acceptez les nouvelles clés GPG des dépôts si nécessaire.

Démarrez et activez **gorgoned**:

```shell
systemctl start gorgoned
systemctl enable gorgoned
```

### Actions post montée de version

Du fait du nouveau format de configuration du module Broker de Engine, la
configuration doit être re-déployée.

Déployer la configuration du Poller depuis l'interface web en suivant
[cette procedure](../monitoring/monitoring-servers/deploying-a-configuration), et en choisissant la méthode
*Redémarrer* pour le processus Engine

## Migrer Centreon Poller Display vers Remote Server

Si la plateforme a des Collecteurs avec le module Poller Display, reférez vous à la procédure
[Migration d'une plate-forme avec Poller Display](../migrate/poller-display-to-remote-server).

## Communications

Par défaut, la communication entre le serveur Central et les Pollers ou les
Remote Servers utilisera toujours SSH.

Considérez changer le protocole de communication en suivant la procédure
[Changer la communication de SSH à ZMQ](../monitoring/monitoring-servers/communications#changer-la-communication-de-ssh-a-zmq).

## Sécurisez votre plateforme

N'oubliez pas de sécuriser votre plateforme Centreon en suivant nos
[recommandations](../administration/secure-platform)
