# Howto setup central authentication with OpenLDAP, SSSD and Rex

In this tutorial i will show how you can setup OpenLDAP and SSSD for a central authentication solution automated with Rex.

Rex (http://www.rexify.org/) is the acronym for Remote Execution. Rex is a framework for Configuration- and Deployment-Management. Rex doesn't need an agent on the hosts you want to manage. Everything is done via ssh.

OpenLDAP (http://www.openldap.org/) is an open source directory server widely used for account management. It is easy to setup and administrate. There are also some Webfrontends like http://phpldapadmin.sourceforge.net/wiki/index.php/Main_Page and https://www.ldap-account-manager.org/lamcms/.

SSSD (https://fedorahosted.org/sssd/) is the acronym for System Security Services Daemon. With its help it is possible to to authenticate your linux users against an OpenLDAP directory server with some nifty additions like offline support.



## Dependencies

During the time of writing the Rex LDAP Module is located here: https://bitbucket.org/jfried/rex-recipes/src/fca0623457780009695b29d3fcdb5637d5bcea9c/Rex/LDAP/OpenLDAP/?at=feature/openldap
