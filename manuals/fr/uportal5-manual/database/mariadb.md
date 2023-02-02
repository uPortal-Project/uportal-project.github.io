# Configuration avec une Base de Données MariaDB

En exemple fonctionnel vous pouvez regarder les fichiers de configuration des test travis dans `uPortal-start/.travis/conf/database/mariadb/`

## Étape 1 : Paramétrage du server MariaDB
Editer le fichier /etc/mysql/mariadb.conf.d/60-server.cnf. (ici pour Debian 9)
Dans la partie `[mysqld]` ajouter les éléments suivant :

```properties
default-storage-engine=INNODB
lower_case_table_names=1
innodb-large-prefix=1
innodb_file_format=Barracuda
innodb_file_format_check=1
innodb_file_format_max=Barracuda
innodb_file_per_table=1
innodb_strict_mode=ON

innodb_buffer_pool_size=2G
innodb_data_home_dir=/var/lib/mysql/
innodb_data_file_path=ibdata1:100M:autoextend
innodb_flush_log_at_trx_commit=1
innodb_log_file_size=256M
innodb_log_buffer_size=64M
```

**NOTE:** À partir de mariaDB 10.1.35 indiquer en plus la configuration suivante:
```properties
innodb_default_row_format=dynamic
```
Cela aura pour effet de créer par défaut toutes les tables avec le row_format=dynamic s'il n'est pas indiqué.

En complément vous pouvez indiquer ces propriétés afin de définir l'UTF-8 par défaut, cela est facultatif si vous créez votre base de données avec le jeu de caractères `uft8mb4` et la collation adequat (cf ci après).
```properties
character-set-server  = utf8mb4
collation-server      = utf8mb4_unicode_520_ci
```

## Étape 2 : Configurer l'utilisateur et la base de donnée

Se connecter au serveur de base de données.
```SQL
CREATE USER 'uportal'@'localhost' IDENTIFIED BY 'uportal';
create database uportal CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci;
GRANT ALL PRIVILEGES ON uportal.* TO 'portail'@'localhost';
# If you want to install portlets on a specific database
# create database portlet CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci;
# GRANT ALL PRIVILEGES ON portlets.* TO 'portail'@'localhost';
```

Avec MariaDB et MySQL le jeu de caractère par défaut doit être défini à `utf8mb4` au lieu de `utf8` car l'encodage UTF-8 de MySQL n'est qu'un support sur 3 octets (3-bytes) de l'encodage unicode.
La partie sur 3 octets n'est pas un support complet de l'UTF-8, cela ne supportera pas les caractères Asiatiques ainsi que les émoticônes. [Regarder ici pour plus de détails](https://dev.mysql.com/doc/refman/5.5/en/charset-unicode.html)

Aussi la collation `utf8mb4_unicode_520_ci` est un nouvel et bon algorithme pour ordonner les données en UTF-8, mais vous pouvez tout aussi bien rester sur 'utf8_unicode_ci' [Regarder la documentation MySQL pour les détails](https://dev.mysql.com/doc/refman/5.6/en/charset-collation-names.html)

## Étape 3 : Configurer Uportal 

### Éditer uPortal-start/gradle.properties 
```properties
mysqldbVersion=5.1.45
```
### Éditer uPortal-start/overlays/build.gradle
```gradle
dependencies {
        /*
         * Add additional JDBC driver jars to the 'jdbc' configuration below;
         * do not remove the hsqldb driver jar that is already listed.
         */
        jdbc "org.hsqldb:hsqldb:${hsqldbVersion}"
        jdbc "mysql:mysql-connector-java:${mysqldbVersion}"

```

### Éditer uPortal-start/etc/portal/global.properties 

Dans la partie Database Connection
```properties
hibernate.connection.driver_class=com.mysql.jdbc.Driver
hibernate.connection.url=jdbc:mysql://localhost/portlets
hibernate.connection.username=uportal
hibernate.connection.password=uportal
hibernate.connection.validationQuery=select 1
hibernate.dialect = org.hibernate.dialect.MySQL5InnoDBDialect
```

Vous devez copier/coller cette configuration pour chaque personnalisation d'accès à la base de données des contextes portlets / uPortal [cf configuration générale des bases de données](README.md#Étape-5-configuration-spécifique-portlet--uportal-optionnel)

**NOTE:** Avant mariaDB 10.1.35 il fallait utiliser le dialect `org.apereo.portal.utils.MySQL5InnoDBCompressedDialect` si vous n'aviez pas configuré votre serveur mariaDB avec le [row_format par défaut ou équivalent](mariadb.md#Étape-1--paramétrage-du-server-mariadb).

**NOTE:** Selon la version d'hibernate utilisée et la version du serveur de données il peut être nécessaire de sélectionner un Dialect adéquat [voici où chercher](https://github.com/hibernate/hibernate-orm/tree/main/hibernate-core/src/main/java/org/hibernate/dialect) (Attention à sélectionner la bonne version en fonction du tag)


## Étape 4 : Initialisation de la Base de Donnée
```shell
./gradlew dataInit
```
## Étape 5 : Déploiement de uPortal
```shell
./gradlew tomcatDeploy
```
