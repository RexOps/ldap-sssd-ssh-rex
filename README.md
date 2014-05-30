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

## Exploring the Rexfile

Now i will guide you through the *Rexfile* so that you can modify it to your needs and setup your OpenLDAP/SSSD infrastructure.

### Loading all required libraries

If you open the *Rexfile* you'll see at the top some ```use``` commands.

```perl
use Rex -base;
```

This loads the base functions of Rex, so that you can use them inside the Rexfile.

```perl
use Rex::LDAP::OpenLDAP;
use Rex::LDAP::OpenLDAP::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Client;
use Rex::LDAP::OpenLDAP::UserManagement::Server;
```

This loads all the LDAP functions we're going to use to setup the OpenLDAP server and the authentication on the clients.


### Authentication

You also need to tell Rex how it can login to your servers. Rex supports password and key authentication.

```perl
user "root";
password "box";
```

Here Rex is taught to login as user *root* with the password *box*. If you want to use key based authentication you can use something like this.

```perl
user "root";
private_key "/home/your-user/.ssh/id_rsa";
public_key "/home/your-user/.ssh/id_rsa.pub";
```

### Group your server

Rex also needs to know on which server it should connect. So you can define server groups.

```perl
group server => "auth01";
group client => "auth01", "fe[01..10]";
```


### Setup OpenLDAP

To setup OpenLDAP it is important to follow one rule. Never ever try to manage the configuration files inside /etc/openldap/slapd.d with an editor or by writing to the files directly. Sooner or later this will get you into trouble. So always use ldapmodify/ldapadd command to change things inside this directory.

With Rex you can use *ldap_entry* resource to manage these entries.

The default installation of OpenLDAP is not able to manage SSH Keys inside it. But there is a schema we will add to it later so that this is possible, too.

If you consider to put your OpenLDAP installation to production i recommend you to read http://www.openldap.org/doc/admin24/replication.html how to setup replication.

This tutorial also do not cover the access control management. Please read http://www.openldap.org/doc/admin24/access-control.html for more information on this topic.

In the *Rexfile* you'll find the task *setup_server*.

```perl
task "setup_server",
  group => "server",
  make {
```

This task is configured to run on all servers registered in the group *server*.

```perl
  Rex::LDAP::OpenLDAP::setup {
    ldap_admin_password         => 'admin',
    ldap_root_dn                => 'dc=rexify,dc=org',
    ldap_root_dn_admin_password => 'test',
    ldap_configure_tls          => TRUE,
  };

  Rex::LDAP::OpenLDAP::UserManagement::Server::add_ssh_public_key;

```

First it install OpenLDAP and create a root database for you. It also configures the admin password of your LDAP server.

*ldap_root_dn* is the name of the database Rex should create for you. Normaly you want to add your organizations domain. Something like *dc=company,dc=tld*.

*ldap_root_db_admin_password* is the password for the admin account of your database. This will be *cn=admin,dc=rexify,dc=org*.

*ldap_configure_tls* can be set to *TRUE* or *FALSE*. If you set it to true, you have to add the SSL cert-, key- and ca file into the folder *files/openldap/certs*, so that Rex can upload them to the LDAP server.

The second function call ```add_ssh_public_key``` will add the SSH-key schema to the OpenLDAP server, so that it is possible to add the public ssh keys into OpenLDAP.

#### Populating OpenLDAP

Now you have a working OpenLDAP you need to populate it with some data.

First we need to create a default folder structure inside it, so you can manage your groups and users. For this you can use the ```ldap_entry``` resource. This resource ensures that the entry exists and has the options set that are defined.

```perl
ldap_entry "ou=People,dc=rexify,dc=org",
  ensure      => 'present',
  objectClass => [ 'top', 'organizationalUnit' ],
  ou          => 'People';

ldap_entry "ou=Groups,dc=rexify,dc=org",
  ensure      => 'present',
  objectClass => [ 'top', 'organizationalUnit' ],
  ou          => 'Groups';

ldap_entry "ou=Services,dc=rexify,dc=org",
  ensure      => 'present',
  objectClass => [ 'top', 'organizationalUnit' ],
  ou          => 'Services';
```

This code creates 3 *folders* (the LDAP name for such a *folder* is *organizationalUnit*, hence the acronym *ou*). One for the user accounts *ou=People,dc=rexify,dc=org*, one for the groups *ou=Groups,dc=rexify,dc=org* and one for special service accounts *ou=Services,dc=rexify,dc=org*.

You can create you own structure this is just an example.

Then you need to create a service user for sssd. SSSD use this user to search the LDAP database to find the user that tries to login.

```perl
ldap_account "sssd",
  ensure        => 'present',
  dn            => 'ou=Services,dc=rexify,dc=org',
  givenName     => 'SSSD',
  sn            => 'Service User',
  uidNumber     => '4000',
  gidNumber     => 0,
  loginShell    => '/bin/false',
  homeDirectory => '/tmp',
  userPassword  => 'abcdef';
```

Now lets create a sample group and user.

```perl
ldap_group "ldapusers",
  ensure    => 'present',
  dn        => 'ou=Groups,dc=rexify,dc=org',
  gidNumber => 3000;

ldap_account "sampleuser",
  ensure        => 'present',
  dn            => 'ou=People,dc=rexify,dc=org',
  givenName     => 'SampleUser',
  sn            => 'Surename',
  uidNumber     => 5000,
  gidNumber     => 3000,
  homeDirectory => '/home/sampleuser',
  loginShell    => '/bin/bash',
  mail          => 'sample.user@gmail.com',
  userPassword  => '{CRYPT}vPYgtKD.j9iL2',
  sshPublicKey  => 'ssh-rsa AAAAB3NzaC1y...',
  groups => ['cn=ldapusers,ou=Groups,dc=rexify,dc=org'];
```

This will create a group names *ldapusers* inside the organizational unit (ou) *ou=Groups,dc=rexify,dc=org* and a user *sampleuser* inside *ou=People,dc=rexify,dc=org*. You can create the crypted password string with the tool *slapdpasswd*.

## Setup SSSD


## Exploring the code

The Rexfile

The other files
