# RH-SSO SSL Quickstarts

The quickstarts demonstrate securing applications with RH-SSO using end-to-end SSL. They provide small, specific, working examples
that can be used as a reference for your own project.  The expectation is that the keycloak server, 
the application server, and the applications are all behind SSL transports


Introduction
------------

These quickstarts run on Red Hat JBoss Enterprise Application Platform 6.4 (currently untested) or 7.

Prior to running the quickstarts you should read this entire document and have completed the following steps:

* [Start and configure the RH-SSO Server](#rh-sso)
* [Start and configure the JBoss EAP Server](#jboss-eap)
* [Configure the Java Certificate Store](#java-certificate)

Afterwards you should read the README file for the quickstart you would like to deploy. See [examples](#examples) for
a list of the available quickstarts.

If you run into any problems please refer to the [troubleshooting](#troubleshooting) section.


Use of RHSSO_HOME and EAP_HOME Variables
-----------------------------------------

The quickstart README files use the replaceable value RHSSO_HOME to denote the path to the RH SSO installation and the
value EAP_HOME to denote the path to the JBoss EAP installation. When you encounter this value in a README file, be sure
to replace it with the actual path to your installations.


System Requirements
-------------------

The applications these projects produce are designed to be run on Red Hat JBoss Enterprise Application Platform 6.4 or 7.

All you need to build these projects is Java 8.0 (Java SDK 1.8) or later and Maven 3.1.1 or later.


Maven Repository
------------------

If you need to build the quickstarts without access to the Red Hat repository then you need the RH-SSO maven repository and the 
EAP repository based on the EAP version you're using.


<a id="rh-sso"></a>Start the RH-SSO Server
------------------------------------------

By default the RH-SSO Server uses the same ports as the JBoss EAP Server. To run the quickstarts you can either run the
 RH-SSO Server on a separate host (machine, VM, Docker, etc..) or on different ports.

To start the RH-SSO server on a separate host:

1. Open a terminal on the separate machine and navigate to the root of the RH-SSO server directory.

2. The following shows the command to start the RH-SSO server:

   ````
   For Linux:   RHSSO_HOME/bin/standalone.sh -b 0.0.0.0
   For Windows: RHSSO_HOME\bin\standalone.bat -b 0.0.0.0
   ````

3. The URL of the RH-SSO server will be http://&lt;HOSTNAME&gt;:8080 (replace &lt;HOSTNAME&gt; with the hostname of the separate host).

To start the RH-SSO server on different ports:

1. Open a terminal and navigate to the root of the RH-SSO server directory.

2. The following shows the command to start the RH-SSO server:

   ````
   For Linux:   RHSSO_HOME/bin/standalone.sh -Djboss.socket.binding.port-offset=100
   For Windows: RHSSO_HOME\bin\standalone.bat -Djboss.socket.binding.port-offset=100
   ````

3. The URL of the RH-SSO server will be *http://localhost:8180*

### <a id="rh-sso-certificate"></a>Create and Deploy the RH-SSO Certificate

1. Create the JKS certificate for the RH-SSO EAP Server.
   ````
   $ keytool -genkey -alias localhost -keyalg RSA -keystore keycloak.jks -validity 10950
       Enter keystore password: secret
       Re-enter new password: secret
       What is your first and last name?
       [Unknown]:  localhost
       What is the name of your organizational unit?
       [Unknown]:  Keycloak
       What is the name of your organization?
       [Unknown]:  Red Hat
       What is the name of your City or Locality?
       [Unknown]:  Westford
       What is the name of your State or Province?
       [Unknown]:  MA
       What is the two-letter country code for this unit?
       [Unknown]:  US
       Is CN=localhost, OU=Keycloak, O=Test, L=Westford, ST=MA, C=US correct?
       [no]:  yes
   ````

2. Export the JKS
   ````
   keytool -exportcert -alias localhost -rfc -keystore keycloak.jks -file keycloak.crt
   ````
### <a id="add-admin"></a>Add Admin User

Open the main page for the RH-SSO server ([localhost:8180](http://localhost:8180) or http://&lt;HOSTNAME&gt;:8080). If
this is a new installation of RH-SSO server you will be instructed to create an initial admin user. To continue with
the quickstarts you need to do this prior to continuing.

### <a id="rh-sso-ssl"></a>Configure RH-SSO for SSL
Adapted from the Keycloak Server Installation and Configuration Guide, section 7.3 HTTPS/SSL Setup. 
https://keycloak.gitbooks.io/server-installation-and-configuration/content/v/1.9/topics/network/https.html

1. Configure Keycloak to Use the Keystore by adding a new security realm. In the standalone or domain configuration file, 
search for the security-realms element and add:

   ````
   <security-realm name="UndertowRealm">
       <server-identities>
           <ssl>
               <keystore path="keycloak.jks" relative-to="jboss.server.config.dir" keystore-password="secret" />
           </ssl>
       </server-identities>
   </security-realm>
   ````
2. Find the element server name="default-server" (it’s a child element of subsystem xmlns="urn:jboss:domain:undertow:) 
and add the https-listener:

   ````
   <subsystem xmlns="urn:jboss:domain:undertow:3.0">
      <buffer-cache name="default"/>
      <server name="default-server">
         <https-listener name="https" socket-binding="https" security-realm="UndertowRealm"/>
      ...
   </subsystem>
   ````


### <a id="add-roles-user"></a>Create Roles and User

To be able to use the examples you need to create some roles as well as at least one sample user. To do first this open
the RH-SSO admin console ([localhost:8180/auth/admin](https://localhost:8543/auth/admin) or http://&lt;HOSTNAME&gt;:8080/auth/admin) and
login with the admin user you created in the [add admin user](#add-admin) section.

Start by creating a user role:

* Select `Roles` from the menu
* Click `Add Role`
* Enter `user` as `Role Name`
* Click `Save`

Next create a user:

* Select `Users` from the menu
* Click `Add user`
* Enter any values you want for the user
* Click `Save`
* Select `Credentials` from the tabs
* Enter a password in `New Password` and `Password Confirmation`
* Click on the toggle to disable `Temporary`
* Click `Reset Password`
* Click `Role Mappings`
* Select `user` under `Available Roles` and click `Add selected`

As an alternative to manually creating the role and user you can use the partial import feature in the admin console and import
the file [config/partial-import.json](config/partial-import.json) into your realm.

One more step, if you want to access the examples with the admin user you need to add the `user` role to admin user:

* Select `Users` from the menu
* Click `View all users`
* Click `Edit` for admin user
* Click `Role Mappings`
* Select `user` under `Available Roles` and click `Add selected`




<a id="jboss-eap"></a>Start and Configure the JBoss EAP Server
--------------------------------------------------------------

Before starting the JBoss EAP server start by extracting the RH-SSO client adapter into it.

For JBoss EAP 7 extract `RH-SSO-7.0.0.GA-eap7-adapter.zip` into EAP_HOME and for JBoss EAP 6.4 extract
`RH-SSO-7.0.0.GA-eap6-adapter.zip` into EAP_HOME. 

If you plan to try the SAML examples you also need the SAML JBoss EAP adapter. To do this for JBoss EAP 7 extract
`RH-SSO-7.0.0.GA-saml-eap7-adapter.zip` into EAP_HOME and for JBoss EAP 6.4 extract
`RH-SSO-7.0.0.GA-saml-eap6-adapter.zip` into EAP_HOME.

The next step is to start JBoss EAP server:

1. Open a terminal and navigate to the root of the JBoss EAP server directory.
2. Use the following command to start the JBoss EAP server:
   ````
   For Linux:   EAP_HOME/bin/standalone.sh
   For Windows: EAP_HOME\bin\standalone.bat
   ````
3. To install the RH-SSO adapter run the following commands:
   ````
   For Linux:

     EAP_HOME/bin/jboss-cli.sh -c --file=EAP_HOME/bin/adapter-install.cli
     EAP_HOME/bin/jboss-cli.sh -c --command=:reload

   For Windows:

    EAP_HOME\bin\jboss-cli.bat -c --file=EAP_HOME\bin\adapter-install.cli
    EAP_HOME\bin\jboss-cli.bat -c --command=:reload
   ````
4. If you plan to try the SAML examples you also need to install RH SSO SAML adapter:

   ````
   For Linux:

     EAP_HOME/bin/jboss-cli.sh -c --file=EAP_HOME/bin/adapter-install-saml.cli
     EAP_HOME/bin/jboss-cli.sh -c --command=:reload

   For Windows:

     EAP_HOME\bin\jboss-cli.bat -c --file=EAP_HOME\bin\adapter-install-saml.cli
     EAP_HOME\bin\jboss-cli.bat -c --command=:reload
   ````

### <a id="jboss-eap-alias"></a>Alias the JBoss EAP Server Hostname

1. Edit your hosts file so that the 127.0.0.1 localhost entry also contains 'appserver'.
e.g. "127.0.0.1		localhost.localdomain localhost appserver"

### <a id="jboss-eap-certificate"></a>Create and Deploy the EAP Certificate
Adapted from the Red Hat JBoss Enterprise Application Platform Documentation:
https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/7.0/paged/how-to-configure-server-security/chapter-2-securing-the-server-and-its-interfaces
These steps are similar to the securing the RH-SSO EAP Server, since both servers are EAP.  This is just another approach,
and it is shown here for you to see different ways to handle the security.

1. Create the JKS certificate for the EAP Application Server.

   ````
   $ keytool -genkeypair -alias appserver -storetype jks -keyalg RSA -keysize 2048 -keypass password1 -keystore EAP_HOME/standalone/configuration/identity.jks -storepass password1 -dname "CN=appserver,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v
   ````

2. Secure the management interface. To ensure the management interfaces bind to HTTPS, you must add the management-https 
configuration and remove the management-http configuration. Use the following CLI commands to bind the management interfaces to HTTPS:

   ````
   /core-service=management/management-interface=http-interface:write-attribute(name=secure-socket-binding, value=management-https)
   
   /core-service=management/management-interface=http-interface:undefine-attribute(name=socket-binding)
   ````

3. Create a New Security Realm. 

Create a properties file named https-mgmt-users.properties located in the EAP_HOME/standalone/configuration/ directory for storing usernames and passwords.
   ````
   $ touch EAP_HOME/standalone/configuration/https-mgmt-users.properties
   ````
Next, enter the following CLI commands to create a new security realm named ManagementRealmHTTPS: 
   ````
    /core-service=management/security-realm=ManagementRealmHTTPS:add
    
    /core-service=management/security-realm=ManagementRealmHTTPS/authentication=properties:add(path=https-mgmt-users.properties,relative-to=jboss.server.config.dir)
   ````
Add the admin user to the new realm.
   ````
   $ EAP_HOME/bin/add-user.sh -up EAP_HOME/standalone/configuration/https-mgmt-users.properties -r ManagementRealmHTTPS
   ...
   Enter the details of the new user to add.
   Using realm 'ManagementRealmHTTPS' as specified on the command line.
   ...
   Username : httpUser
   Password requirements are listed below. To modify these restrictions edit the add-user.properties configuration file.
    - The password must not be one of the following restricted values {root, admin, administrator}
    - The password must contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
    - The password must be different from the username
   ...
   Password :
   Re-enter Password :
   About to add user 'httpUser' for realm 'ManagementRealmHTTPS'
   ...
   Is this correct yes/no? yes
   ..
   Added user 'httpUser' to file 'EAP_HOME/configuration/https-mgmt-users.properties'
   ...
   Is this new user going to be used for one AS process to connect to another AS process?
   e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server EJB calls.
   yes/no? no
   ````
Configure the management interface to use the new realm.
   ````
   /core-service=management/management-interface=http-interface:write-attribute(name=security-realm,value=ManagementRealmHTTPS)
   ````
Configure the security realm to use the new keystore.
   ````
   /core-service=management/security-realm=ManagementRealmHTTPS/server-identity=ssl:add(keystore-path=identity.jks,keystore-relative-to=jboss.server.config.dir,keystore-password=password1, alias=appserver)
   ````
Reload the server.
   ````
   reload
   ````
   
Update jboss-cli.xml to connect to new management port.
   ````
   <jboss-cli xmlns="urn:jboss:cli:2.0">
       <default-protocol use-legacy-override="true">https-remoting</default-protocol>
       <!-- The default controller to connect to when 'connect' command is executed w/o arguments -->
       <default-controller>
           <protocol>https-remoting</protocol>
           <host>localhost</host>
           <port>9993</port>
       </default-controller>eee712585
       ccccccfnuhhrdblfbhcenfvbbfdkfctlufkffftvjjub
       
   ````
   
Add and configure an HTTPS Security Realm to the Undertow webserver.
   ````
   /core-service=management/security-realm=HTTPSRealm/:add
   ````
   ````
   /core-service=management/security-realm=HTTPSRealm/server-identity= \
   ssl:add(keystore-path=identity.jks, \
   keystore-relative-to=jboss.server.config.dir, \
   keystore-password=password1, alias=appserver)
   ````

Update the Undertow Subsystem to use the HTTPS Security Realm
Once an HTTPS security realm has been configured, an https-listener needs to be configured in the Undertow subsystem which references that security realm:
   ````
   /subsystem=undertow/server=default-server/https-listener=https:add( \
   socket-binding=https, security-realm=HTTPSRealm)
   ````
   
### <a id="jboss-eap-certificate"></a>Enable SSL

1. Login to RH-SSO Auth admin: 
https://localhost:8543/auth/admin
2. Navigate to Realm Settings -> Login Tab
3. Select "all requests" for Require SSL.
4. Click Save.


### <a id="jboss-eap-certificate"></a>Secure the Administration Console

<a id="java-certificate"></a>Configure the Java Certificate Store
--------------------------------------------------------------
//TODO not needed if we configure two way truststores in EAP

Examples
--------

* [app-jee-html5](app-jee-html5/README.md) - HTML5 application that invokes the example service. Requires service example to be deployed.
* [app-jee-jsp](app-jee-jsp/README.md) - JSP application packaged that invokes the example service. Requires service example to be deployed.
* [app-profile-jee-html5](app-profile-jee-html5/README.md) - HTML5 application that displays user profile and token details.
* [app-profile-jee-jsp](app-profile-jee-jsp/README.md) - JSP application that displays user profile and token details.
* [app-profile-jee-vanilla](app-profile-jee-vanilla/README.md) - JSP application configured with basic authentication. Shows how to secure an application with the client adapter subsystem.
* [app-profile-saml-jee-jsp](app-profile-saml-jee-jsp/README.md) - JSP application that uses SAML and displays user profile.
* [service-jee-jaxrs](service-jee-jaxrs/README.md) - JAX-RS Service with public and protected endpoints.


Troubleshooting
---------------

| Problem | Probable Cause | Possible Solution |
|---------|----------------|-------------------|
| Some required files are missing / Some Enforcer rules have failed | Client adapter config is missing | Add client adapter installation file to `config` directory as specified in quickstart README.md |
| Unknown authentication mechanism KEYCLOAK | OpenID Connect client adapter missing | Install OpenID Connect adapter as specified in the [Start and Configure the JBoss EAP Server](#jboss-eap) section |
| Unknown authentication mechanism KEYCLOAK-SAML | SAML client adapter missing | Install SAML adapter as specified in the [Start and Configure the JBoss EAP Server](#jboss-eap) section |
| Failed to invoke service: 404 Not Found | Service not deployed, or service URL not correct | Deploy service or change the URL for the service as specified in the quickstart README
| Failed to invoke service: Request failed message with no error code | CORS not enabled | Most likely cause is that you've deployed the HTML5 application to a different host than the service, if so the solution is to add CORS support to the service. See the README for the service for how to enable. |
| Page displays: Forbidden | Authenticated user is missing a role required to access the url | This can happen if you fail to add `user` role to admin user as instructed in [Create Roles and User](#add-roles-user). |
