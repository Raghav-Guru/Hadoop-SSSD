**Install and configure SSSD on Hadoop cluster hosts**

Following steps details about configuring sssd on all Node Managers in a secure clusters to fetch the user info while a job is submitted. If this SSSD is not desired you may would need to create and manager users locally in all node managers.Following procedure is followed is verified in one of the cluster hosts: 

**Step 1:** Install sssd pkgs: 

    #yum install sssd sssd-clients

**Step 2:** Configured sssd.conf with below config:

    #vi /etc/sssd/sssd.conf
    [sssd]
    config_file_version = 2
    reconnection_retries = 3
    sbus_timeout = 30
    services = nss, pam
    domains = AD
    
    [domain/AD]
    description = AD 
    #debug_level = 2
    min_id = 1000
    id_provider = ldap
    auth_provider = ldap
    ldap_schema = rfc2307bis
    cache_credentials = true
     
    ###Update the AD server and other details ###
     
    ldap_search_base = <Base>
    ldap_uri = ldaps://<AD_server>
    ldap_default_bind_dn = <BindDN>
    ldap_default_authtok_type = password
    ldap_default_authtok = <Bind Password>
     
    #ldap_id_use_start_tls = true
    ldap_tls_reqcert = allow
    ldap_tls_cacertdir = /etc/openldap/certs
    ldap_user_object_class = user
    ldap_user_name = sAMAccountName
    ldap_user_principal = userPrincipalName
    ldap_group_object_class = group
    ldap_group_name = cn
    ldap_force_upper_case_realm = True
     
    ##UID/GID Mapping config 
    ldap_user_objectsid = objectSid
    ldap_group_objectsid = objectSid
    ldap_user_primary_group = primaryGroupID
    case_sensitive = false
    ldap_id_mapping = true
    fallback_homedir = /home/%u
    default_shell = /bin/bash
     
    [nss]
    reconnection_retries = 3
    #debug_level = 2
     
    [pam]
    reconnection_retries = 3
    #debug_level = 2

**Step 3:** Import SSL certs of AD server to the cacerts db under /etc/openldap/certs:

    #”echo | openssl s_client -connect <AD-Server>:636 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /var/tmp/ad.crt”
    #cd /etc/openldap/certs
    #certutil –A –d . –n “AD cert” –t “C,,” –i /var/tmp/ad.crt

**Step 4:** Restart SSSD and verify if “id –a <AD User> shows the ad user info: 

    #service sssd restart
    #id –a <AD-User>

**Step 5:** Once verified we have to configure PAM to allow AD users to login: 

    #yum install authconfig
    #authconfig --enablesssdauth --update

Login to the host with AD accounts and verify the access to node. As sssd does caching of users on first start, sssd startup may take time as it has to connect to AD to get the user and cache it. 


