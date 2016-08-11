# WSO2 Enterprise Store Puppet Module

This repository contains the Puppet Module for installing and configuring WSO2 Enterprise Store 2.0.0 on various environments. Configuration data is managed using [Hiera](http://docs.puppetlabs.com/hiera/1/). Hiera provides a mechanism for separating configuration data from Puppet scripts and managing them in a set of YAML files in a hierarchical manner.

## Supported Operating Systems

- Debian 6 or higher
- Ubuntu 12.04 or higher

## Supported Puppet Versions

- Puppet 2.7, 3 or newer

## How to Contribute
Follow the steps mentioned in the [wiki](https://github.com/wso2/puppet-base/wiki) to setup a development environment and update/create new puppet modules.

## Add WSO2 Base Puppet Module

Run the following commands to get wso2base submodule.

````
git submodule init
git submodule update
````

## Packs to be Copied

Copy the following files to their corresponding locations.

1. WSO2 Enterprise Store distribution (2.0.0) to `<PUPPET_HOME>/modules/wso2es/files`
2. JDK 1.7_80 distribution to `<PUPPET_HOME>/modules/wso2base/files`

## Running WSO2 Enterprise Store in the `default` profile
No changes to Hiera data are required to run the `default` profile.  Copy the above mentioned files to their corresponding locations and apply the Puppet Modules.

## Running WSO2 Enterprise Store with clustering in specific profiles
Hiera data sets matching the distributed profiles of WSO2 Enterprise Store (`publisher`, `store`) are shipped with clustering related configuration already enabled. Therefore, only a few changes are needed to setup a distributed deployment. For more details refer the [WSO2 Enterprise Store clustering guide](https://docs.wso2.com/display/CLUSTER44x/Clustering+ES+2.0.0)

1. If the Clustering Membership Scheme is `WKA`, add the Well Known Address list.

   Ex:
    ```yaml
    wso2::clustering:
      enabled: true
      local_member_host: "%{::ipaddress}"
      local_member_port: 4000
      domain: es.wso2.domain
      membership_scheme: wka
      wka:
        members:
          -
            hostname: 192.168.100.82
            port: 4000
          -
            hostname: 192.168.100.81
            port: 4000
    ```

2. Uncomment and modify the MySQL based data sources to point to the external MySQL servers.

   Ex:
    ```yaml
    wso2::master_datasources:
      wso2_config_db:
        name: WSO2_CONFIG_DB
        description: The datasource used for config registry
        driver_class_name: "%{hiera('wso2::datasources::mysql::driver_class_name')}"
        url: jdbc:mysql://192.168.100.1:3306/ES_CONFIG_DB?autoReconnect=true
        username: "%{hiera('wso2::datasources::mysql::username')}"
        password: "%{hiera('wso2::datasources::mysql::password')}"
        jndi_config: jdbc/WSO2_CONFIG_DB
        max_active: "%{hiera('wso2::datasources::common::max_active')}"
        max_wait: "%{hiera('wso2::datasources::common::max_wait')}"
        test_on_borrow: "%{hiera('wso2::datasources::common::test_on_borrow')}"
        default_auto_commit: "%{hiera('wso2::datasources::common::default_auto_commit')}"
        validation_query: "%{hiera('wso2::datasources::mysql::validation_query')}"
        validation_interval: "%{hiera('wso2::datasources::common::validation_interval')}"
        
    ```

3. Uncomment (and optionally configure) registry mounting

   Ex:
    ```yaml
    wso2::registry_mounts:
      wso2_config_db:
        path: /_system/config
        target_path: /_system/config/es
        read_only: false
        registry_root: /
        enable_cache: true

      wso2_gov_db:
        path: /_system/governance
        target_path: /_system/governance
        read_only: false
        registry_root: /
        enable_cache: true
    ```

## Running WSO2 Enterprise Store with Secure Vault
WSO2 Carbon products may contain sensitive information such as passwords in configuration files. [WSO2 Secure Vault](https://docs.wso2.com/display/Carbon444/Securing+Passwords+in+Configuration+Files) provides a solution for securing such information.

Uncomment and modify the below changes in Hiera file to apply Secure Vault.

1. Enable Secure Vault

    ```yaml
    wso2::enable_secure_vault: true
    ```

2. Add Secure Vault configurations as below

    ```yaml
    wso2::secure_vault_configs:
      <secure_vault_config_name>:
        secret_alias: <secret_alias>
        secret_alias_value: <secret_alias_value>
        password: <password>
    ```

    Ex:
    ```yaml
    wso2::secure_vault_configs:
      key_store_password:
        secret_alias: Carbon.Security.KeyStore.Password
        secret_alias_value: repository/conf/carbon.xml//Server/Security/KeyStore/Password,false
        password: wso2carbon
    ```

3. Add Cipher Tool configuration file templates to `template_list`

    ```yaml
    wso2::template_list:
      - repository/conf/security/cipher-text.properties
      - repository/conf/security/cipher-tool.properties
      - bin/ciphertool.sh
    ```

    Please add the `password-tmp` template also to `template_list` if the `vm_type` is not `docker` when you are running the server in `default` platform.

## Running WSO2 Enterprise Store on Kubernetes
WSO2 Puppet Module ships Hiera data required to deploy WSO2 Enterprise Store on Kubernetes. For more information refer to the documentation on [deploying WSO2 products on Kubernetes using WSO2 Puppet Modules](https://docs.wso2.com/display/PM210/Deploying+WSO2+Products+on+Kubernetes+Using+WSO2+Puppet+Modules).
