[//]: # (title: Automate Server Installation)
[//]: # (auxiliary-id: Automate Server Installation)

For automated server installation, use the `.tar.gz` distribution.

Typically, you will need to unpack it and make the script perform the steps noted in the [Configuring Server for Production Use](#Configuring+Server+for+Production+Use) section.

If you want to get a preconfigured server right away, put files from a previously configured server into the Data Directory. For each new server you will need to ensure it points to a new database (configured in `<Data Directory>\config\database.properties`) and change `<Data Directory>\config\main-config.xml` file not to have the `uuid` attribute in the root XML element (so new one can be generated) and setting appropriate value for "rootURL" attribute.

