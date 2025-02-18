---
title: Installation options
---

You can install Search Services using the distribution zip (with or without mutual TLS).

## Install with mutual TLS (zip)

Use this information to install Search Services on the same machine as Alfresco Content Services with mutual TLS.

Mutual TLS is used for authentication between the Repository and Search Services.

This task assumes you have:

* Installed Alfresco Content Services 6.0 or above
* Set the following properties in the `<TOMCAT_HOME>/shared/classes/alfresco-global.properties` file:

    ```text
    index.subsystem.name=solr6
    solr.secureComms=https
    solr.port.ssl=8983
    ```

> **Important:** Alfresco strongly recommends you use firewalls and other infrastructure means to ensure the Search Services server is not accessible from anything other than trusted hosts and/or users, and only on the ports needed for Search Services.

1. Download `alfresco-search-services-1.4.x.zip` from the [Alfresco Support Portal](https://support.alfresco.com/){:target="_blank"} if you are an Alfresco Content Services Enterprise user, or from [Alfresco Community Edition](https://www.alfresco.com/products/community/download){:target="_blank"} if you are an Alfresco Content Services Community user.

2. Extract the Search Services distribution.

    By default, the contents of `alfresco-search-services-1.4.x.zip` are decompressed in a root folder as `/alfresco-search-services`. See [Search Services directory structure]({% link search-services/1.4/config/index.md %}#search-services-directory-structure) for more details.

3. If you use several languages across your organization, you **must** enable cross-language search support in all fields. To do this update the `alfresco-search-services/solrhome/conf/shared.properties` file:

    ```bash
    alfresco.cross.locale.datatype.0={http://www.alfresco.org/model/dictionary/1.0}text
    alfresco.cross.locale.datatype.1={http://www.alfresco.org/model/dictionary/1.0}content
    alfresco.cross.locale.datatype.2={http://www.alfresco.org/model/dictionary/1.0}mltext
    ```

4. (Optional) Suggestion is disabled by default. To enable suggestion update the `alfresco-search-services/solrhome/conf/shared.properties` file.

    ```bash
    alfresco.suggestable.property.0={http://www.alfresco.org/model/content/1.0}name
    alfresco.suggestable.property.1={http://www.alfresco.org/model/content/1.0}title
    alfresco.suggestable.property.2={http://www.alfresco.org/model/content/1.0}description
    alfresco.suggestable.property.3={http://www.alfresco.org/model/content/1.0}content
    ```

    > **Note:** The spell check functionality does not work with Search Services when suggestion is enabled.

5. To secure access to Search Services, you must create a new set of keystores and keys.

    1. Generate secure keys specific to your Alfresco installation. For more information, see [Secure keys]({% link search-services/1.4/config/keys.md %}#generate-secure-keys-for-ssl-communication).

    2. Create a new keystore directory at `alfresco-search-services/solrhome`.

    3. In the production environment, copy your custom keystore and truststore to the `alfresco-search-services/solrhome/keystore` directory.

    4. Update the SSL-related system properties by replacing `<SOLR_HOME> with alfresco-search-services/solrhome`, and set your keystore and truststore passwords.

        (Windows) update the `alfresco-search-services/solr.in.cmd` file:

        ```bash
        set SOLR_SSL_KEY_STORE=<SOLR_HOME>\keystore\ssl.repo.client.keystore
        set SOLR_SSL_KEY_STORE_PASSWORD=password
        set SOLR_SSL_TRUST_STORE=<SOLR_HOME>\keystore\ssl.repo.client.truststore
        set SOLR_SSL_TRUST_STORE_PASSWORD=password
        set SOLR_SSL_NEED_CLIENT_AUTH=true
        set SOLR_SSL_WANT_CLIENT_AUTH=false
        ```

        (Linux) update the `alfresco-search-services/solr.in.sh` file:

        ```bash
       SOLR_SSL_KEY_STORE=<SOLR_HOME>/keystore/ssl.repo.client.keystore
       SOLR_SSL_KEY_STORE_PASSWORD=password
       SOLR_SSL_TRUST_STORE=<SOLR_HOME>/keystore/ssl.repo.client.truststore
       SOLR_SSL_TRUST_STORE_PASSWORD=password 
       SOLR_SSL_NEED_CLIENT_AUTH=true 
       SOLR_SSL_WANT_CLIENT_AUTH=false
       ```

    5. Set the `SOLR_PORT` environment variable:

        (Windows) update the `alfresco-search-services/solr.in.cmd` file:

        ```bash
        set SOLR_PORT=8983
        ```

        (Linux) update the `alfresco-search-services/solr.in.sh` file:

        ```bash
        SOLR_PORT=8983
        ```

6. (Optional) If you want to install Search Services on a separate machine, set the `SOLR_SOLR_HOST` and `SOLR_ALFRESCO_HOST` environment variables before starting Search Services, for more see [Configuring Search Services]({% link search-services/1.4/config/index.md %}#search-services-externalized-configuration).

    (Windows) update the `alfresco-search-services/solr.in.cmd` file:

    ```bash
    set SOLR_SOLR_HOST=localhost
    ```

    ```bash
    set SOLR_ALFRESCO_HOST=localhost
    ```

    (Linux) update the `alfresco-search-services/solr.in.sh` file:

    ```bash
    SOLR_SOLR_HOST=localhost
    ```

    ```bash
    SOLR_ALFRESCO_HOST=localhost
    ```

7. To configure the Solr6 cores, set the following:

    * Before creating the alfresco and archive cores:
        * Set `alfresco.secureComms=https` in `alfresco-search-services/solrhome/templates/rerank/conf/solrcore.properties`.
        * Copy the custom keystores to the `alfresco-search-services/solrhome/keystore` directory.

            ```bash
            ssl-repo-client.keystore
            ssl-repo-client.truststore
            ssl-keystore-passwords.properties
            ssl-truststore-passwords.properties
            ```

    * If the alfresco and archive cores already exist, ensure that `alfresco.secureComms` is set to `https` for both the cores. For example:
        * `alfresco-search-services/solrhome/alfresco/conf/solrcore.properties`
        * `alfresco-search-services/solrhome/archive/conf/solrcore.properties`
8. For running a single instance of Search Services (i.e. not sharded), use the following commands:

   > **Note:** You should run the application as a dedicated user. For example, you can create a Solr user.

    ```bash
    cd alfresco-search-services
    ./solr/bin/solr start -a "-Djavax.net.ssl.keyStoreType=JCEKS -Djavax.net.ssl.trustStoreType=JCEKS -Dsolr.ssl.checkPeerName=false -Dcreate.alfresco.defaults=alfresco,archive"
    ```

    > **Note:** The `-Dcreate.alfresco.defaults=alfresco,archive` command automatically creates the `alfresco` and `archive` cores. Therefore, you should only start Search Services with `-Dcreate.alfresco.defaults=alfresco,archive` the first time you run Search Services. In addition, to ensure that Search Services connects using the IPv6 protocol instead of IPv4, add `-Djava.net.preferIPv6Addresses=true` to the startup parameters.

    The default port used is 8983.

    The command line parameter, `-a` passes additional JVM parameters, for example, system properties using `-D`.

    Once Search Services is up and running, you should see a message like:

    ```text
    Waiting up to 180 seconds to see Solr running on port 8983 []  
    Started Solr server on port 8983 (pid=24289). Happy searching!
    ```

    To stop all instances of Search Services, use:

    ```bash
    ./solr/bin/solr stop
    ```

    The logs are stored in the `alfresco-search-services/logs/solr.log` file, by default. This can be configured in `solr.in.sh` (for Linux) or `solr.in.cmd` (for Windows) using `SOLR_LOGS_DIR`.

    You have successfully created an `alfresco` core and an `archive` core. To verify, in a browser, navigate to the Solr URL, [https://localhost:8983/solr](https://localhost:8983/solr). 

    In the Solr Admin UI, select the core selector drop-down list and verify that both the `alfresco` and `archive` cores are present.

    Allow a few minutes for Search Services to start indexing.

    > **Note:** The Admin Console is only available when you are using Alfresco Content Services Enterprise.

## Install without mutual TLS (zip)

Use this information to install Search Services on the same machine as Alfresco Content Services without mutual TLS.

Mutual TLS is used for authentication between the Repository and Search Services. Without mutual TLS, internal APIs on both sides will be exposed without any form of authentication, giving full access to the repository data. In such a setup, you need to make sure that external access to these APIs is blocked, for example, with a front-end reverse proxy. See [Adding a reverse proxy in front of Content Services]({% link content-services/6.1/install/zip/tomcat.md %}#adding-a-reverse-proxy-in-front-of-content-services)
 for more.

This task assumes you have:

* Installed Alfresco Content Services 6.0 or above
* Set the following properties in the `<TOMCAT_HOME>/shared/classes/alfresco-global.properties` file:

    ```text
    index.subsystem.name=solr6
    solr.secureComms=none
    solr.port=8983
    ```

> **Important:** Alfresco strongly recommends you use firewalls and other infrastructure means to ensure the Search Services server is not accessible from anything other than trusted hosts and/or users, and only on the ports needed for Search Services.

1. Download `alfresco-search-services-1.4.x.zip` from the [Alfresco Support Portal](https://support.alfresco.com/){:target="_blank"} if you are an Alfresco Content Services Enterprise user, or from [Alfresco Community Edition](https://www.alfresco.com/products/community/download){:target="_blank"} if you are an Alfresco Content Services Community user.

2. Extract the Search Services distribution.

    By default, the contents of `alfresco-search-services-1.4.x.zip` are decompressed in a root folder as `/alfresco-search-services`. See [Search Services directory structure]({% link search-services/1.4/config/index.md %}#search-services-directory-structure) for more details.

3. Configure HTTP.

    1. Open `solrhome/templates/rerank/conf/solrcore.properties`.

    2. Replace `alfresco.secureComms=https` with:

        ```bash
        alfresco.secureComms=none
        ```

        This ensures that the Solr cores are created in plain HTTP mode.

        Alternatively, you can add this configuration in the system properties (using `-D`) when starting Solr. For example, add the following to the startup parameters in step 7.

        ```bash
        -Dalfresco.secureComms=none
        ```

4. If you use several languages across your organization, you **must** enable cross-language search support in all fields. To do this add the following to the `alfresco-search-services/solrhome/conf/shared.properties` file:

    ```bash
    alfresco.cross.locale.datatype.0={http://www.alfresco.org/model/dictionary/1.0}text
    alfresco.cross.locale.datatype.1={http://www.alfresco.org/model/dictionary/1.0}content
    alfresco.cross.locale.datatype.2={http://www.alfresco.org/model/dictionary/1.0}mltext
    ```

5. (Optional) Suggestion is disabled by default. To enable suggestion update the `alfresco-search-services/solrhome/conf/shared.properties` file.

    ```bash
    alfresco.suggestable.property.0={http://www.alfresco.org/model/content/1.0}name
    alfresco.suggestable.property.1={http://www.alfresco.org/model/content/1.0}title
    alfresco.suggestable.property.2={http://www.alfresco.org/model/content/1.0}description
    alfresco.suggestable.property.3={http://www.alfresco.org/model/content/1.0}content
    ```

    > **Note:** The spell check functionality works with Search Services when suggestion is enabled.

6. (Optional) If you want to install Search Services on a separate machine, set the `SOLR_SOLR_HOST` and `SOLR_ALFRESCO_HOST` environment variables before starting Search Services, for more see [Configuring Search Services]({% link search-services/1.4/config/index.md %}#search-services-externalized-configuration).

    (Windows) update the `alfresco-search-services`/`solr.in.cmd` file:

    ```bash
    set SOLR_SOLR_HOST=localhost
    ```

    ```bash
    set SOLR_ALFRESCO_HOST=localhost
    ```

    (Linux) update the alfresco-search-services/solr.in.sh file:

    ```bash
    SOLR_SOLR_HOST=localhost
    ```

    ```bash
    SOLR_ALFRESCO_HOST=localhost
    ```

7. To start Search Services with all the default settings, use the following command:

    ```bash
    ./solr/bin/solr start -a "-Dcreate.alfresco.defaults=alfresco,archive"
    ```

    The command line parameter, `-a` passes additional JVM parameters, for example, system properties using `-D`.

    > **Note:** The `-Dcreate.alfresco.defaults=alfresco,archive` command automatically creates the `alfresco` and `archive` cores. Therefore, you should only start Search Services with `-Dcreate.alfresco.defaults=alfresco,archive` the first time you run Search Services.
    > **Note:** You should run this application as a dedicated user. For example, you can create a Solr user.
    > **Note:** To ensure that Search Services connects using the IPv6 protocol instead of IPv4, add `-Djava.net.preferIPv6Addresses=true` to the startup parameters.

    Once Search Services is up and running, you should see a message similar to the following:

    ```bash
    Waiting up to 180 seconds to see Solr running on port 8983 []  
    Started Solr server on port 8983 (pid=24289). Happy searching!
    ```

    To stop the currently running Search Services instance, use:

    ```bash
    ./solr/bin/solr stop
    ```

    The logs are stored in the `alfresco-search-services/logs/solr.log` file, by default. This can be configured in `solr.in.sh` (for Linux) or `solr.in.cmd` (for Windows) using `SOLR_LOGS_DIR`.

    You have successfully created an `alfresco` core and an `archive` core. To verify, in a browser, navigate to the Solr URL, [http://localhost:8983/solr](http://localhost:8983/solr). In the Solr Admin UI, select the core selector drop-down list and verify that both the `alfresco` and `archive` cores are present.

    Allow a few minutes for Search Services to start indexing.

8. Go to **Admin Console > Repository Services > Search Service** and verify that:

    1. You see the Solr 6 option in the **Search Service In Use** list.

    2. Under **Main (Workspace) Store Tracking Status**, the **Approx Transactions to Index** is **0**.
