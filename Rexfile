use Rex -base;
use Data::Dumper;
use Rex::LDAP::OpenLDAP;
use Rex::LDAP::OpenLDAP::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Commands;
use Rex::LDAP::OpenLDAP::UserManagement::Client;
use Rex::LDAP::OpenLDAP::UserManagement::Server;

user "root";
password "box";

group server => "10.211.55.168";

set openldap => {
  bind_dn  => 'cn=admin,dc=rexify,dc=org',
  password => 'test',
  base_dn  => 'dc=rexify,dc=org',
};

task "setup_client",
  group => "server",
  make {

  Rex::LDAP::OpenLDAP::UserManagement::Client::setup {
    ldap_base_dn       => 'dc=rexify,dc=org',
    ldap_uri           => 'ldaps://10.211.55.168',
    ldap_bind_dn       => 'cn=nss,ou=Services,dc=rexify,dc=org',
    ldap_bind_password => 'abcdef',
    ldap_base_user_dn  => 'ou=People,dc=rexify,dc=org',
    ldap_base_group_dn => 'ou=Groups,dc=rexify,dc=org',
    configure_ssh_ldap => TRUE,
  };

  };

task "setup_server",
  group => "server",
  make {

  Rex::LDAP::OpenLDAP::setup {
    ldap_admin_password         => 'admin',
    ldap_root_dn                => 'dc=rexify,dc=org',
    ldap_root_dn_admin_password => 'test',
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

  ldap_entry "ou=HostGroup,dc=rexify,dc=org",
    ensure      => 'present',
    objectClass => [ 'top', 'organizationalUnit' ],
    ou          => 'HostGroup';

  ldap_entry "cn=admin,ou=HostGroup,dc=rexify,dc=org",
    ensure      => 'present',
    cn          => 'admin',
    objectClass => [ 'top', 'groupOfNames' ],
    member      => ['cn=jfried,ou=People,dc=rexify,dc=org'];

  ldap_group "ldapusers",
    ensure    => 'present',
    dn        => 'ou=Groups,dc=rexify,dc=org',
    gidNumber => 3000;

  ldap_account "nss",
    ensure        => 'present',
    dn            => 'ou=Services,dc=rexify,dc=org',
    givenName     => 'nss',
    sn            => 'account',
    uidNumber     => '4000',
    gidNumber     => 0,
    loginShell    => '/bin/false',
    homeDirectory => '/tmp',
    userPassword  => 'abcdef';

  ldap_account "jfried",
    ensure        => 'present',
    dn            => 'ou=People,dc=rexify,dc=org',
    givenName     => 'Jan',
    sn            => 'Gehring',
    uidNumber     => '5000',
    gidNumber     => 3000,
    homeDirectory => '/home/jfried',
    loginShell    => '/bin/bash',
    mail          => 'jan.gehring@gmail.com',
    userPassword  => '{CRYPT}vPYgtKD.j9iL2',
    sshPublicKey =>
    'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQChUwh/HH1NGFB7M3m9DyL/hH3HoJpRu9mi52JIUI2PhZhBU/idp3qcvhur2U9IpwQfjjzgei7IN3dgTjnIB1CFG9ux4PJ56gS2NYvcHd+FsuSJecGLEkYamq2dk2+CgRC7rPGLhxCrBO29FI9O5Lfew8hiFOVIIIY2S9xXgWXQeH/bxqrsJW/WoGusZjZaWCJkYZmC7y3CvxswBkDOAXdf8tl1rbRQbTjRSiEoCQjsCFYV3unNDS7gwSf2a9nBZmBkWibr69HbofE/LScKauhasXz55bqCRdn+ELJiHTJfi4tExRwhHxl6mtD1VjfxBfts5gJgdRclOpni1nuwPPIp jan@pitahaya.local',
    groups => ['cn=ldapusers,ou=Groups,dc=rexify,dc=org'];

  ldap_account "ferki",
    ensure        => 'present',
    dn            => 'ou=People,dc=rexify,dc=org',
    givenName     => 'Ferenc',
    sn            => 'Erki',
    uidNumber     => '5001',
    gidNumber     => 0,
    homeDirectory => '/home/ferki',
    loginShell    => '/bin/bash',
    mail          => 'ferki@rexify.org',
    userPassword  => 'dddfsdd',
    groups        => ['cn=ldapusers,ou=Groups,dc=rexify,dc=org'];


  };
