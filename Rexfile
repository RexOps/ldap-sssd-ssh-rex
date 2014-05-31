use Rex -base;

use Rex::LDAP::OpenLDAP;
use Rex::LDAP::OpenLDAP::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Client;
use Rex::LDAP::OpenLDAP::UserManagement::Server;

user "root";
password "box";

group server => "10.211.55.168";
group client => "10.211.55.168", "10.211.55.169";

set openldap => {
  bind_dn  => 'cn=admin,dc=rexify,dc=org',
  password => 'test',
  base_dn  => 'dc=rexify,dc=org',
};

task "setup_server",
  group => "server",
  make {

  Rex::LDAP::OpenLDAP::setup {
    ldap_admin_password         => 'admin',
    ldap_base_dn                => 'dc=rexify,dc=org',
    ldap_base_dn_admin_password => 'test',
    ldap_configure_tls          => TRUE,
  };

  Rex::LDAP::OpenLDAP::UserManagement::Server::add_ssh_public_key;

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

  ldap_account "sssd",
    ensure        => 'present',
    dn            => 'ou=Services,dc=rexify,dc=org',
    givenName     => 'sssd',
    sn            => 'Service Account',
    uidNumber     => 4000,
    gidNumber     => 1,
    loginShell    => '/bin/false',
    homeDirectory => '/tmp',
    userPassword  => 'abcdef';

  ldap_group "ldapusers",
    ensure    => 'present',
    dn        => 'ou=Groups,dc=rexify,dc=org',
    gidNumber => 3000;

  ldap_account "jfried",
    ensure        => 'present',
    dn            => 'ou=People,dc=rexify,dc=org',
    givenName     => 'Jan',
    sn            => 'Gehring',
    uidNumber     => 5000,
    gidNumber     => 3000,
    homeDirectory => '/home/jfried',
    loginShell    => '/bin/bash',
    mail          => 'jan.gehring@gmail.com',
    userPassword  => '{CRYPT}vPYgtKD.j9iL2',
    sshPublicKey  => 'ssh-rsa AAAAB3NzaC1....',
    groups        => ['cn=ldapusers,ou=Groups,dc=rexify,dc=org'];

  };

task "setup_client",
  group => "client",
  make {

  Rex::LDAP::OpenLDAP::UserManagement::Client::setup {
    ldap_base_dn       => 'dc=rexify,dc=org',
    ldap_uri           => 'ldaps://10.211.55.168',
    ldap_bind_dn       => 'cn=sssd,ou=Services,dc=rexify,dc=org',
    ldap_bind_password => 'abcdef',
    ldap_base_user_dn  => 'ou=People,dc=rexify,dc=org',
    ldap_base_group_dn => 'ou=Groups,dc=rexify,dc=org',
    configure_ssh_ldap => TRUE,
  };

  };
