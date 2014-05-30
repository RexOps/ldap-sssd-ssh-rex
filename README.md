# Howto setup central authentication with OpenLDAP, SSSD and Rex

In this tutorial i will show how you can setup OpenLDAP and SSSD for a central authentication solution automated with Rex.

Rex (http://www.rexify.org/) is the acronym for Remote Execution. Rex is a framework for Configuration- and Deployment-Management. Rex doesn't need an agent on the hosts you want to manage. Everything is done via ssh.

OpenLDAP (http://www.openldap.org/) is an open source directory server widely used for account management. It is easy to setup and administrate. There are also some Webfrontends like http://phpldapadmin.sourceforge.net/wiki/index.php/Main_Page and https://www.ldap-account-manager.org/lamcms/.

SSSD (https://fedorahosted.org/sssd/) is the acronym for System Security Services Daemon. With its help it is possible to to authenticate your linux users against an OpenLDAP directory server with some nifty additions like offline support.

## Preparation

For this tutorial i've used CentOS 6. To follow the tutorial you need one system where you can install Rex (this can also be installed on your Workstation), one system for OpenLDAP and one (or some) client systems to configure pam to authenticate against OpenLDAP.


## Installing Rex

The installation of Rex is easy. If you're using Linux you may find packages for it on http://www.rexify.org/get/.

If there is no package for your distribution available, you can also install it via CPAN. For this you have to install libssh2-devel first.

```bash
sudo sh -c "curl -L cpanmin.us | perl - Rex"
```

## Project Setup

To initialize your project you can use the ```rexify``` command to download the code and all dependencies.

```bash
rexify --init=https://bitbucket.org/jfried/ldap-sssd-ssh-rex.git
```

This command will download the code and all dependencies into the folder *ldap-sssd-ssh-rex*.

## Setup OpenLDAP

To setup OpenLDAP it is important to follow one rule. Never ever try to manage the configuration files inside /etc/openldap/slapd.d with an editor or by writing to the files directly. Sooner or later this will get you into trouble. So always use ldapmodify/ldapadd command to change things inside this directory.

With Rex you can use *ldap_entry* resource to manage these entries.

The default installation of OpenLDAP is not able to manage SSH Keys inside it. But there is a schema we will add to it later so that this is possible, too.

If you consider to put your OpenLDAP installation to production i recommend you to read http://www.openldap.org/doc/admin24/replication.html how to setup replication.



## Setup SSSD


## Exploring the code

The Rexfile

The other files
