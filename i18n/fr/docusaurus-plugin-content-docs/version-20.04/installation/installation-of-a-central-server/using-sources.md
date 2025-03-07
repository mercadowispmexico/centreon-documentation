---
id: using-sources
title: A partir des sources
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## Prérequis

> La plupart des utilisateurs de CentOS préfèreront installer Centreon Web en utilisant
[les paquets fournis par Centreon](using-packages).

<Tabs groupId="sync">
<TabItem value="Redhat/CentOS" label="Redhat/CentOS">

Afin d’installer les logiciels Centreon, le dépôt Software collections de Red Hat doit être activé.

> Le dépôt Software collections est nécessaire pour l’installation de PHP 7 et les librairies associées.

Exécutez la commande suivante :

``` shell
yum install -y centos-release-scl
```

Le dépôt est maintenant installé.

Vous pouvez maintenant installer les prérequis :

``` shell
yum update
yum install -y \
    rh-php72
    rh-php72-php-zip \
    rh-php72-php-xml \
    rh-php72-php-fpm \
    rh-php72-php-process \
    rh-php72-php-common \
    rh-php72-php-pdo \
    rh-php72-php-intl \
    rh-php72-php-pear \
    rh-php72-php-json \
    rh-php72-php-mysqlnd \
    rh-php72-php-ldap \
    rh-php72-php-gd \
    rh-php72-php-cli \
    rh-php72-php-mbstring \
    rh-php72-php-snmp \
    openssl \
    perl-DBD-MySQL \
    perl-Sys-Syslog \
    httpd24-httpd \
    perl-DBI \
    perl-DBD-MySQL \
    rrdtool \
    perl-rrdtool \
    perl-Crypt-DES \
    perl-Digest-SHA1 \
    perl-Digest-HMAC \
    net-snmp-utils \
    perl-Socket6 \
    perl-IO-Socket-INET6 \
    net-snmp \
    net-snmp-libs \
    dmidecode \
    lm_sensors \
    net-snmp-perl \
    fping \
    cpp \
    gcc \
    gcc-c++ \
    libstdc++ \
    glib2-devel
```

Des commandes additionnelles sont nécessaires pour configurer correctement l'environnement :

``` shell
usermod -U apache
/opt/rh/rh-php72/root/bin/pear channel-update pear.php.net
```

Si vous ne pouvez pas accéder directement à Internet directement mais passer par un proxy, exécutez la commande
suivante :

``` shell
/opt/rh/rh-php72/root/bin/pear config-set http_proxy http://my_proxy.com:port
```

Puis exécutez :

``` shell
/opt/rh/rh-php72/root/bin/pear upgrade-all
```

</TabItem>
<TabItem value="Debian Stretch / Ubuntu 18.04" label="Debian Stretch / Ubuntu 18.04">

Ajoutez le dépot suivant, nécéssaire pour installer php 7.2 :

### For Debian Stretch

``` shell
apt-get install wget apt-transport-https lsb-release ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" >> /etc/apt/sources.list.d/php.list
apt-get update
```

### For Ubuntu 18.04

> Il est nécessaire d'ajouter sudo devant les commandes ci-dessous :
>
> ``` shell  
> apt-get install software-properties-common
> add-apt-repository ppa:ondrej/php
> apt update
> ```

Installez les dépendances nécessaires :

``` shell
apt-get install \
    php7.2 \
    php7.2-opcache \
    libapache2-mod-php7.2 \
    php7.2-mysql \
    php7.2-curl \
    php7.2-json \
    php7.2-gd \
    php7.2-mcrypt \
    php7.2-intl \
    php7.2-mbstring \
    php7.2-xml \
    php7.2-zip \
    php7.2-fpm \
    php7.2-readline \
    php7.2-sqlite3 \
    php7.2-ldap \
    php7.2-snmp \
    php-db \
    php-date
    php-pear \
    sudo \
    tofrodos \
    bsd-mailx \
    lsb-release \
    mariadb-server \
    libconfig-inifiles-perl \
    libcrypt-des-perl \
    libdigest-hmac-perl \
    libdigest-sha-perl \
    libgd-perl
```

Activez les modules :

``` shell
a2enmod proxy_fcgi setenvif proxy rewrite
a2enconf php7.2-fpm
a2dismod php7.2
systemctl restart apache2 php7.2-fpm
```

Des commandes additionnelles sont nécessaires pour configurer correctement l'environnement :

``` shell
groupadd -g 6000 centreon
useradd -u 6000 -g centreon -m -r -d /var/lib/centreon -c "Centreon Admin" -s /bin/sh centreon
```

Pour finir, vous devez installer des MIBs SNMP. En raison d'un problème de licence, les fichiers MIBs ne sont pas
disponibles par défaut sous Debian. Pour les ajouter, modifiez le fichier */etc/apt/sources.list* et ajoutez la
catégorie **non-free**.

Puis exécutez les commandes suivantes :

``` shell
apt-get update
apt-get install snmp-mibs-downloader
```

Modifiez le fichier de configuration SNMP */etc/default/snmpd* en ajoutant :

``` shell
export MIBDIRS=/usr/share/snmp/mibs
export MIBS=ALL
```

Commentez :

``` shell
#mibs ALL
```

Redémarrez le service SNMP :

``` shell
service snmpd restart
service snmptrapd restart
```

</TabItem>
<TabItem value="Suse" label="Suse">

Installez les dépendances nécessaires :

``` shell
yast -i gcc \
    gcc-c++ \
    make \
    automake \
    apache2 \
    php5 \
    php5-mysql \
    apache2-mod_php5 \
    php5-pear \
    php5-ldap \
    php5-snmp \
    php5-gd \
    php5-soap \
    php5-intl \
    php5-posix \
    php5-gettext \
    php5-mbstring \
    mysql \
    libmysqlclient-devel \
    perl-DBD-mysql \
    mysql-community-server \
    rrdtool \
    perl-Config-IniFiles \
    net-snmp \
    perl-Net-SNMP \
    perl-SNMP \
    gd \
    libjpeg-devel \
    libpng-devel \
    fontconfig-devel \
    freetype2-devel \
    sudo \
    mailx \
    fping \
    iputils \
    dos2unix \
    cron \
    dejavu \
    nagios-plugins
```

Sur certaines distributions OpenSuse, le paramétrage par défaut des type **mine** n'est pas valide pour fonctionner
avec l'interface web Centreon. Editez le fichier */etc/mime.types* et rechercher les lignes :

``` shell
text/x-xsl xsl
text/x-xslt xslt xsl
```

Remplacez-les par :

``` shell
text/xml xsl
text/xml xslt xsl
```

Sauvegardez le fichier et redémarrez apache :

``` shell
/etc/init.d/apache2 restart
```

</TabItem>
</Tabs>

## Moteur de supervision (Centreon Engine)

Installez ce moteur avant de poursuivre l'installation. N'oubliez pas d'installer les
[Nagios plugins](https://library.nagios.com/) si vous ne l'avez pas déjà fait.

## Multiplexeur de flux (Centreon Broker)

Installez ce multiplexeurs de flux avant de poursuivre l'installation.

## Centreon

Téléchargez la dernière version de [Centreon ici](https://download.centreon.com).

Extraire Centreon de l'archive :

``` shell
tar zxf centreon-web-YY.MM.x.tar.gz
cd centreon-web-YY.MM.x
```

> Le script d'installation permet une configuration personnalisée, cette procédure vous montrera les meilleurs chemins
> à utiliser.
>
> En outre, les questions rapides Yes/No peuvent être répondues par [y] la plupart du temps.
>
> Si les sources de centreon ont été téléchargées depuis github, exécutez ces commandes :
>
> ``` shell
> composer install --no-dev --optimize-autoloader
> npm install
> npm run build
> ```

Exécutez le script d'installation :

``` shell
./install.sh -i
```

### Contrôle de prérequis

> Si l'étape d'installation des prérequis s'est déroulée avec succès, vous ne devriez avoir aucun problème lors de cette
> étape. Sinon, reprennez la procédure d'installation des prérequis.

``` shell
###############################################################################
#                                                                             #
#                         Centreon (www.centreon.com)                         #
#                          Thanks for using Centreon                          #
#                                                                             #
#                                    vYY.MM.x                                 #
#                                                                             #
#                              infos@centreon.com                             #
#                                                                             #
#                   Make sure you have installed and configured               #
#         centreon-gorgone - sudo - sed - php - apache - rrdtool - mysql      #
#                                                                             #
###############################################################################
------------------------------------------------------------------------
        Checking all needed binaries
------------------------------------------------------------------------
rm                                                         OK
cp                                                         OK
mv                                                         OK
/bin/chmod                                                 OK
/bin/chown                                                 OK
echo                                                       OK
more                                                       OK
mkdir                                                      OK
find                                                       OK
/bin/grep                                                  OK
/bin/cat                                                   OK
/bin/sed                                                   OK

------------------------------------------------------------------------
        Check mandatory gorgone service status
------------------------------------------------------------------------

Is the Gorgone module already installed?
[y/n], default to [n]:
> y
```

### Approbation de la licence

``` shell
This General Public License does not permit incorporating your program into
proprietary programs.  If your program is a subroutine library, you may
consider it more useful to permit linking proprietary applications with the
library.  If this is what you want to do, use the GNU Library General
Public License instead of this License.

Do you accept GPLv2 license ?
[y/n], default to [n]:
> y
```

### Composants principaux

Répondez **[y]** à toutes les questions :

``` shell
------------------------------------------------------------------------
        Please choose what you want to install
------------------------------------------------------------------------

Do you want to install : Centreon Web Front
[y/n], default to [n]:
> y

Do you want to install : Centreon Nagios Plugins
[y/n], default to [n]:
> y

Do you want to install : CentreonTrapd process
[y/n], default to [n]:
> y
```

### Définition des chemins d'installation

``` shell
------------------------------------------------------------------------
        Start CentWeb Installation
------------------------------------------------------------------------

Where is your Centreon directory ?
default to [/usr/local/centreon]
> /usr/share/centreon
Path /usr/share/centreon                                   OK

Where is your Centreon log directory ?
default to [/usr/local/centreon/log]
> /var/log/centreon

Do you want me to create this directory ? [/var/log/centreon]
[y/n], default to [n]:
> y
Path /var/log/centreon                                     OK

Where is your Centreon etc directory ?
default to [/etc/centreon]
>

Do you want me to create this directory ? [/etc/centreon]
[y/n], default to [n]:
> y
Path /etc/centreon                                         OK

Where is your Centreon variable state information directory ?
default to [/var/lib/centreon]
>

Do you want me to create this directory ? [/var/lib/centreon]
[y/n], default to [n]:
> y
Path /var/lib/centreon                                     OK

Where is rrdtool
default to [/usr/bin/rrdtool]
> /opt/rrdtool-broker/bin/rrdtool
/opt/rrdtool-broker/bin/rrdtool                            OK

/usr/bin/mail                                              OK

Where is your php binary ?
default to [/usr/bin/php]
>
/usr/bin/php                                               OK

Where is PEAR [PEAR.php]
default to [/usr/share/pear/PEAR.php]
> /usr/share/php/PEAR.php
Path /usr/share/php/PEAR.php                               OK
/usr/bin/perl                                              OK
Composer dependencies are installed                        OK
Frontend application is built                              OK
Enable Apache configuration                                OK
Conf centreon already enabled
Finding Apache user :                                      www-data
Finding Apache group :                                     www-data
```

### Utilisateur et groupe centreon

Le groupe d'applications **centreon** est utilisé pour les droits d'accès entre les différents logiciels de la suite
Centreon :

``` shell
What is the Centreon group ? [centreon]
default to [centreon]
>

What is the Centreon user ? [centreon]
default to [centreon]
>
```

### Utilisateur de la supervision

Cet utilisateur exécute le moteur de supervision Centreon Engine.

``` shell
What is the Monitoring engine user ? [centreon-engine]
default to [centreon-engine]
>
```

Cet utilisateur exécute le multiplexeur de flux Centreon Broker.

``` shell
What is your Centreon Broker user ? [centreon-broker]
default to [centreon-broker]
>
```

### Répertoire des journaux d'évènements

``` shell
What is the Monitoring engine log directory ?[/var/log/centreon-engine]
default to [/var/log/centreon-engine]
>
Path                                                       OK
Path                                                       OK
Add group centreon to user www-data                        OK
Add group centreon to user centreon-engine                 OK
Add group centreon-engine to user www-data                 OK
Add group centreon-engine to user centreon                 OK
Add group www-data to user centreon                        OK
```

### Configuration des droits sudo

``` shell
------------------------------------------------------------------------
        Configure Sudo
------------------------------------------------------------------------

Where is sudo configuration file ?
default to [/etc/sudoers.d/centreon]
>

Do you want me to create this file ? [/etc/sudoers.d/centreon]
[y/n], default to [n]:
>  y
/etc/sudoers.d/centreon                                    OK

What is the Monitoring engine binary ? [/usr/sbin/centengine]
default to [/usr/sbin/centengine]
>

Where is the Monitoring engine configuration directory ? [/etc/centreon-engine]
default to [/etc/centreon-engine]
>

Where is the configuration directory for broker module ? [/etc/centreon-broker]
default to [/etc/centreon-broker]
>

Where is your service command binary ?
default to [/usr/sbin/service]
>
Your sudo is not configured

Do you want me to configure your sudo ? (WARNING)
[y/n], default to [n]:
>  y
Configuring Sudo                                           OK
```

### Configuration du serveur Apache

``` shell
------------------------------------------------------------------------
        Configure Apache server
------------------------------------------------------------------------

Do you want to add Centreon Apache sub configuration file ?
[y/n], default to [n]:
> y
Create '/etc/apache2/conf-available/centreon.conf'         OK
Configuring Apache                                         OK

Do you want to reload your Apache ?
[y/n], default to [n]:
> y
Reloading Apache service                                   OK
```

### Configuration de PHP FPM

``` shell
------------------------------------------------------------------------
        Configure PHP FPM service
------------------------------------------------------------------------

Do you want to add Centreon PHP FPM sub configuration file ?
[y/n], default to [n]:
> y
Creating directory /var/lib/centroen/sessions              OK
Create 'etc/php/7.2/fpm/pool.d/centreon.conf'              OK
Configuring PHP FPM                                        OK

Do you want to reload PHP FPM service ?
[y/n], default to [n]:
> y
Reloading PHP FPM service                                  OK
Preparing Centreon temporary files                         OK
Change right on /var/log/centreon                          OK
Change right on /etc/centreon                              OK
Change macros for insertBaseConf.sql                       OK
Change macros for sql update files                         OK
Change macros for php files                                OK
Change macros for php config file                          OK
Change macros for perl binary                              OK
Change right on /etc/centreon-engine                       OK
Add group centreon-broker to user www-data                 OK
Add group centreon-broker to user centreon-engine          OK
Add group centreon to user centreon-broker                 OK
Change right on /etc/centreon-broker                       OK
Copy CentWeb in system directory                           OK
Install CentWeb (web front of centreon)                    OK
Change right for install directory                         OK
Install libraries                                          OK
Write right to Smarty Cache                                OK
Change macros for centreon.cron                            OK
Install Centreon cron.d file                               OK
Change macros for centAcl.php                              OK
Change macros for downtimeManager.php                      OK
Change macros for centreon-backup.pl                       OK
Install cron directory                                     OK
Change right for eventReportBuilder                        OK
Change right for dashboardBuilder                          OK
Change right for centreon-backup.pl                        OK
Change right for centreon-backup-mysql.pl                  OK
Change macros for centreon.logrotate                       OK
Install Centreon logrotate.d file                          OK
Prepare centFillTrapDB                                     OK
Install centFillTrapDB                                     OK
Prepare centreon_trap_send                                 OK
Install centreon_trap_send                                 OK
Prepare centreon_check_perfdata                            OK
Install centreon_check_perfdata                            OK
Prepare centreonSyncPlugins                                OK
Install centreonSyncPlugins                                OK
Prepare centreonSyncArchives                               OK
Install centreonSyncArchives                               OK
Prepare generateSqlLite                                    OK
Install generateSqlLite                                    OK
Install changeRrdDsName.pl                                 OK
Prepare export-mysql-indexes                               OK
Install export-mysql-indexes                               OK
Prepare import-mysql-indexes                               OK
Install import-mysql-indexes                               OK
Prepare clapi binary                                       OK
Install clapi binary                                       OK
Centreon Web Perl lib installed                            OK
```

### Installation des modules pear

``` shell
------------------------------------------------------------------------
Pear Modules
------------------------------------------------------------------------
Check PEAR modules
PEAR                            1.4.9       1.10.8         OK
DB                              1.7.6       1.9.2          OK
Date                            1.4.6       1.4.7          OK
All PEAR modules                                           OK
```

### Installation du fichier de configuration

``` shell
------------------------------------------------------------------------
            Centreon Post Install
------------------------------------------------------------------------
Create /usr/share/centreon/www/install/install.conf.php    OK
Create /etc/centreon/instCentWeb.conf                      OK
```

### Installation du composant Centstorage

``` shell
------------------------------------------------------------------------
        Starting CentStorage Installation
------------------------------------------------------------------------

Where is your Centreon Run Dir directory?
default to [/var/run/centreon]
>

Do you want me to create this directory ? [/var/run/centreon]
[y/n], default to [n]:
> y
Path /var/run/centreon                                     OK

Where is your CentStorage binary directory ?
default to [/usr/share/centreon/bin]
>
Path /usr/share/centreon/bin                               OK

Where is your CentStorage RRD directory ?
default to [/var/lib/centreon]
>
Path /var/lib/centreon                                     OK
Preparing Centreon temporary files
/tmp/centreon-setup exists, it will be moved...
install www/install/createTablesCentstorage.sql            OK
Creating Centreon Directory '/var/lib/centreon/status'     OK
Creating Centreon Directory '/var/lib/centreon/metrics'    OK
Change right : /var/run/centreon                           OK
Install logAnalyserBroker                                  OK
Install nagiosPerfTrace                                    OK
Change macros for centstorage.cron                         OK
Install CentStorage cron                                   OK
Create /etc/centreon/instCentStorage.conf                  OK
```

### Installation des plugins

``` shell
------------------------------------------------------------------------
        Starting Centreon Plugins Installation
------------------------------------------------------------------------
Path                                                       OK
Path                                                       OK

Where is your CentPlugins lib directory
default to [/var/lib/centreon/centplugins]
>

Do you want me to create this directory ? [/var/lib/centreon/centplugins]
[y/n], default to [n]:
> y
Path /var/lib/centreon/centplugins                         OK
Create /etc/centreon/instCentPlugins.conf                  OK
```

### Installation du système de gestion des traps SNMP (Centreontrapd)

``` shell
------------------------------------------------------------------------
        Starting CentreonTrapD Installation
------------------------------------------------------------------------

Path                                                       OK
Path                                                       OK

Where is your SNMP configuration directory ?
default to [/etc/snmp]
>
/etc/snmp                                                  OK

Where is your CentreonTrapd binaries directory ?
default to [/usr/local/centreon/bin]
> /usr/share/centreon/bin
/usr/share/centreon/bin                                    OK

Finding Apache user :                                      www-data
Preparing Centreon temporary files
Change macros for snmptrapd.conf                           OK
Replace CentreonTrapd init script Macro                    OK
Replace CentreonTrapd default script Macro                 OK

Do you want me to install CentreonTrapd init script ?
[y/n], default to [n]:
> y
CentreonTrapd init script installed                        OK
CentreonTrapd default script installed                     OK

Do you want me to install CentreonTrapd run level ?
[y/n], default to [n]:
> y
trapd Perl lib installed                                   OK
Install : snmptrapd.conf                                   OK
Install : centreontrapdforward                             OK
Install : centreontrapd                                    OK
Change macros for centreontrapd.logrotate                  OK
Install Centreon Trapd logrotate.d file                    OK
Create /etc/centreon/instCentPlugins.conf                  OK
```

#### Fin de l'installation

``` shell
###############################################################################
#                                                                             #
#                         Thanks for using Centreon.                          #
#                          -----------------------                            #
#                                                                             #
#                 Go to the URL : http://localhost.localdomain/centreon/      #
#                          to finish the setup                                #
#                                                                             #
#                Please read the documentation available here :               #
#                         documentation.centreon.com                          #
#                                                                             #
#      ------------------------------------------------------------------     #
#                                                                             #
#         Report bugs at https://github.com/centreon/centreon/issues          #
#                                                                             #
#                        Contact : contact@centreon.com                       #
#                          http://www.centreon.com                            #
#                                                                             #
#                          -----------------------                            #
#              For security issues, please read our security policy           #
#              https://github.com/centreon/centreon/security/policy           #
#                                                                             #
###############################################################################
```

### Installation des dépendances PHP

Tout d'abord, vous devez installer l'installeur de dépendance PHP **composer**. Composer peut être téléchargé
[ici](https://getcomposer.org/download/) (celui-ci est également disponible dans les dépôts EPEL).

Une fois que composer est installé, rendez-vous dans les répertoires Centreon (habituellement **/usr/share/centreon/**)
et exécutez la commande suivante :

``` shell
composer install --no-dev --optimize-autoloader
```

### Installation des dépendances Javascript

Tout d'abord, vous devez installer l'environnement d'exécution javascript **nodejs**. Les instructions d'installation
sont disponibles [ici](https://nodejs.org/en/download/package-manager/).

Une fois que nodejs est installé, copiez les fichiers JSON vers le dossier d'installation :

``` shell
cp /usr/local/src/centreon-web-YY.MM.x/package* /usr/share/centreon/
```

Puis, rendez vous dans le répertoire centreon (habituellement **/usr/share/centreon/**) et exécutez les commandes
suivantes :

``` shell
npm install
npm run build
rm -rf node_modules
```

### Pour tous les OS

SELinux doit être désactivé. Pour cela, vous devez modifier le fichier **/etc/sysconfig/selinux** et remplacer
**enforcing** par **disabled** comme dans l'exemple suivant :

``` shell
SELINUX=disabled
```

Après avoir sauvegardé le fichier, veuillez redémarrer votre système d'exploitation pour prendre en compte les changements.

La timezone par défaut ainsi que des paramètres requis de PHP doivent être configurés. Pour cela, allez dans le
répertoire `/etc/php/7.2/cli/conf.d` ou `/etc/php/7.2/apache2/conf.d` et créez un fichier nommé `centreon.ini` contenant
les lignes suivantes :

``` shell
date.timezone = Europe/Paris
max_execution_time = 300
session.use_strict_mode = 1
session.gc_maxlifetime = 7200
```

Après avoir sauvegardé le fichier, n'oubliez pas de redémarrer le service apache de votre serveur.

La base de données MariaDB doit être disponible pour pouvoir continuer l'installation (localement ou non). Pour
information, nous recommandons MariaDB.

## Installation web

Terminez l'installation en réalisant les
[étapes de l'installation web](../web-and-post-installation#installation-web).
