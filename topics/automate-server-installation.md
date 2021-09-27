[//]: # (title: Automate Server Installation)
[//]: # (auxiliary-id: Automate Server Installation)

For an automated TeamCity server installation, use the `.tar.gz` distribution.

Typically, you will need to unpack it and make the script perform the steps noted in [this section](configure-server-installation.md#Configure+Server+for+Production+Use).

If you want to get a preconfigured server right away, put the files from a previously configured server into the [Data Directory](teamcity-data-directory.md). For each new server, you will need to:
* ensure it points to a new database (configured in `<Data Directory>\config\database.properties`);
* change the `<Data Directory>\config\main-config.xml` file not to have the `uuid` attribute in the root XML element (so the new one can be generated); set the appropriate value for the `rootURL` attribute.
