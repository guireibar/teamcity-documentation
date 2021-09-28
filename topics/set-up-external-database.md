[//]: # (title: Set up External Database)
[//]: # (auxiliary-id: Set up External Database;Setting up External Database;Setting up an External Database)

TeamCity stores build history, users, build results, and some runtime data in an SQL database. See the full list [here](manual-backup-and-restore.md).

>If you have been evaluating TeamCity with the internal database and want to copy the data from it, read how to [migrate to an external database](migrate-to-external-database.md).

The currently used database is shown on the __Administration | Global Settings__ page. It is also mentioned in `teamcity-server.log` on the server startup. `HSQL*` means that the internal database is in use.

## Default Internal Database

On the first TeamCity run, using an internal HSQLDB database is suggested by default. The internal database suits evaluation purposes only; it works out of the box and requires no additional setup.

However, __we strongly recommend using an external database__ as a backend TeamCity database in a production environment. An external database is usually more reliable and provides better performance: the internal database may crash and lose all your data (for example, on the "out of disk space" condition). The internal database may become extremely slow on large data sets (database storage files over 200 Mb). Please note that our support does not cover any performance or database data loss issues if you are using the internal database.

In short, __do not use the internal HSQLDB database for production TeamCity instances__. [Migrate to an external database](migrating-to-an-external-database.md) the moment you start to rely on the data stored on your TeamCity server.

## Selecting External Database Engine
[//]: # (AltHead: selectingDatabase)

As a general rule, you should use the database that suits your environment best and that you can better maintain/configure in your organization. While we strive to make sure TeamCity functions equally well under all the [supported databases](supported-platforms-and-environments.md#Databases), issues may surface in some of them under high TeamCity-generated load.

You may also want to estimate the [required database capacity](how-to.md#Estimate+External+Database+Capacity).

## General Steps

1. Configure the external database to be used by TeamCity (see the [database-specific sections](#Database-specific+Steps) below).
2. Configure the connection to the database via the form on the [first TeamCity server start](start-teamcity-server.md) or by configuring the [database connection settings](#Database+Configuration+Properties) manually.    
    Note that TeamCity assumes ownership over the database schema. The database structure is created on the first start and actively modified during the upgrade to the new TeamCity version. The schema is not changed when TeamCity is working normally.  
    The user account used by TeamCity should have __permissions to create new, modify and delete existing tables__ in its schema, in addition to usual __read/write permissions__ on all tables.
3. You may also need to __download the JDBC driver for your database__.   
   Due to licensing terms, TeamCity does not bundle the driver `.jar` files for external databases. You will need to download the Java JDBC driver and put the appropriate `.jar` files (see driver-specific sections below) from it into the `<[TeamCity Data Directory](teamcity-data-directory.md)>/lib/jdbc` directory.    
    Note that the `.jar` files should be compiled for the Java version not greater than the one used to run TeamCity. Otherwise, you might get `Unsupported major.minor version` errors related to the database driver classes.

<anchor name="SettingupanExternalDatabase-DatabasespecificSteps"/>

## Database-specific Steps

The section below describes the required configuration on the database server and the TeamCity server.

### MySQL

#### On MySQL server side

Recommended database server settings:
 * use InnoDB storage engine
 * use [utf8mb4 character set](configuring-utf8-character-set-for-mysql.md) (or utf8 if MySQL version is 5.5.2 or earlier)
 * use case-sensitive collation
 * make sure that the time zone of the JVM running TeamCity and that of MySQL instance are the same using the `my.cnf` file or by configuring time zones at the OS level
 * [see also recommendations for MySQL server settings](how-to.md#Configure+Newly+Installed+MySQL+Server)

The MySQL user account that will be used by TeamCity must be granted all permissions on the TeamCity database. This can be done by executing the following SQL commands from the MySQL console:

```sql
create database <database-name> collate utf8mb4_bin; -- or utf8_bin on MySQL 5.5.2 or earlier
create user <user-name> identified by '<password>';
grant all privileges on <database-name>.* to <user-name>;
grant process on *.* to <user-name>;

```

#### On TeamCity server side (with MySQL)

JDBC driver installation:
1. Download the MySQL [JDBC driver](http://dev.mysql.com/downloads/connector/j/). Make sure you use a version compatible with your server. If the MySQL server version is 5.5 or newer, the JDBC driver version should be at least 5.1.23. For versions above 8, driver version 8 should be used.
2. Place `mysql-connector-java-*-bin.jar` from the downloaded archive into the `<[TeamCity Data Directory](teamcity-data-directory.md)>/lib/jdbc` directory (remove the existing files there, if any). Proceed with the TeamCity setup.

### PostgreSQL

#### On PostgreSQL server side

1. Create an empty database for TeamCity in PostgreSQL.
   * Make sure to set up the database to use UTF8.
   * Grant permissions to modify this database to the user account used by TeamCity to work with the database.
2. [See also recommendations for PostgreSQL server settings](how-to.md#Configure+Newly+Installed+PostgreSQL+Server)

TeamCity does not specify which schema will be used for its tables. By default, PostgreSQL creates tables in the 'public' [schema](https://www.postgresql.org/docs/9.1/static/ddl-schemas.html) ('public' is the name of the schema). TeamCity can also work with other PostgreSQL schemas. To switch to a different schema, do the following:

Create a schema named exactly as the username. This can be done using the `pgAdmin` tool or with the following SQL:

```sql
create schema teamcity authorization teamcity;

```
The schema has to be empty (it must not contain any tables).

#### On TeamCity server side (with PostgreSQL)

Download the required [PostgreSQL JDBC42 driver](http://jdbc.postgresql.org/download.html#current) and place it into the `<[TeamCity Data Directory](teamcity-data-directory.md)>/lib/jdbc` directory (remove the existing files there, if any). Proceed with the TeamCity setup.

### Oracle

#### On Oracle server side
Create an Oracle user account/schema for TeamCity.

>TeamCity uses the primary character set (char, varchar, clob) for storing internal text data and the national character set (nchar2, nvarchar, nclob) to store the user input and data from external systems, like VCS, NTLM, and so on.

* Make sure that the national character set of the database instance is UTF or Unicode.
* Grant the `CREATE SESSION`, `CREATE TABLE` permissions to a user whose account will be used by TeamCity to work with this database.

[//]: # (Internal note. Do not delete. "Setting up an External Databased282e338.txt")    

TeamCity, on the first connect, creates all necessary tables and indices in the user's schema. (Note: TeamCity never attempts to access other schemas even if they are accessible)

<note>

Make sure TeamCity user has quota for accessing table space.
</note>

#### On TeamCity server side (with Oracle)

1. Get the Oracle JDBC driver. Supported driver versions are 11.1 and higher. The Oracle JDBC driver must be compatible with your Oracle server.  
   Place the following files:
   * `ojdbc8.jar` (or `ojdbc6.jar`, `ojdbc7.jar` depending on your database version)
   * `orai18n.jar` (can be omitted if missing in the driver version)  
   into the `<[TeamCity Data Directory](teamcity-data-directory.md)>/lib/jdbc` directory (remove the existing files there, if any).  
  It is strongly recommended locating the driver in your Oracle server installation. Contact your DBA for the files if required. Alternatively, download the Oracle JDBC driver from the [Oracle website](http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html).
2. Proceed with the TeamCity setup.

### Microsoft SQL Server
[//]: # (AltHead: settingUpMSSQL)

For step-by-step instructions, see the [dedicated page](setting-up-teamcity-with-ms-sql-server.md). The current section provides key details required for the setup.

#### On MS SQL server side

>TeamCity uses the primary character set (char, varchar, text) for storing internal text data and the national character set (nchar, nvarchar, ntext) to store the user input and data from external systems, like VCS, NTLM, and so on.

1. Create a new database. As the primary collation, use the case-sensitive collation (collation name ending with `_CS_AS`) corresponding to your locale.
2. Create a TeamCity user and ensure that this user is the owner of the database (grant the user dbo rights), which will give the user the ability to modify the database schema. For SSL connections, ensure that the version of MS SQL server and the TeamCity version of java are compatible. We recommend using the latest update of SQL server.
3. Allocate sufficient transaction log space depending on how intensively the server will be used. The recommended setup is not less than 1Gb.
4. Make sure SQL Server Browser is running.
5. Make sure TCP/IP protocol is enabled for SQL Server instance.

#### On TeamCity server side (with MS SQL)

1. Download the [Microsoft JDBC driver v6.4+](https://www.microsoft.com/en-us/download/details.aspx?id=56615) (`sqljdbc_6.*.x` package) from the Microsoft [Download Center](https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server).
2. Unpack the downloaded package into a temporary directory.
   1. Copy the `mssql-jdbc-*.jre8.jar` from the just downloaded package into the `<[TeamCity Data Directory](teamcity-data-directory.md)>/lib/jdbc` directory (remove the existing files there, if any). MS SQL integrated security (Windows authentication) requires installing `sqljdbc_auth.dll` from the driver package as per [instructions](setting-up-teamcity-with-ms-sql-server.md#integratedSecurityAuth).
3. Proceed with the TeamCity setup.

#### jTDS driver

It is not recommended to use jTDS JDBC driver. There are at least known issues with using Unicode characters when the driver is in use.

If you use the driver (`jtds` text appears in the `connectionUrl` of `database.properties`), it is highly recommended switching the native driver.

The due procedure for the switch is to:
* create the server [backup](teamcity-data-backup.md) including the database
* stop the server and configure the server to use native Microsoft JDBC driver as noted in the section above
* restore the database from the backup into a new MS SQL database
* run the server

## Database Configuration Properties

The database connection settings are stored in the `<[TeamCity Data Directory](teamcity-data-directory.md)>/config/database.properties` file. The file is a Java [properties file](http://en.wikipedia.org/wiki/.properties). You can modify it to specify required properties for your database connections.

For all supported databases there are [template files](teamcity-data-directory.md#.dist+Template+Configuration+Files) with database-specific properties located in the `<[TeamCity Data Directory](teamcity-data-directory.md)>/config` directory. The files have the `database.<database_type>.properties.dist` naming format and can be used as a reference on the required settings.

TeamCity uses Apache DBCP for database connection pooling. Refer to [Apache Commons documentation](http://commons.apache.org/dbcp/configuration.html) for detailed description of configuration properties.

<seealso>
        <category ref="installation">
            <a href="common-problems.md">Common database-related problems</a>
            <a href="migrate-to-external-database.md">Migrating to an External Database</a>
        </category>
        <category ref="concepts">
            <a href="teamcity-data-directory.md">TeamCity Data Directory</a>
        </category>
        <category ref="admin-guide">
            <a href="teamcity-data-backup.md">TeamCity Data Backup</a>
        </category>
</seealso>
