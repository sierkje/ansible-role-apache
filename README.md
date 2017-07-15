Ansible Role: Apache 2.4
=========

An Ansible Role that installs Apache 2.4 on Ubuntu or Debian.

Requirements
------------

If you are using SSL/TLS, you will need to provide your own certificate and key files. You can generate a self-signed certificate with a command like openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout example.key -out example.crt. 

If you are using Apache with PHP, it is recommended to use the [sierkje.apache-php](https://galaxy.ansible.com/sierkje/apache-php) role to connect Apache to PHP via FPM. See that role's README for more info. to connect Apache to PHP via FPM. (The [sierkje.apache-php README file](https://github.com/sierkje/ansible-role-apache-php/blob/master/README.md) has more info.) 

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    apache_listen_ip: "*"
    apache_listen_port: 80
    apache_listen_port_ssl: 443

The IP address and ports on which apache should be listening. Useful if you have another service (like a reverse proxy) listening on port 80 or 443 and need to change the defaults.

    apache_create_vhosts: true
    apache_vhosts_filename: "vhosts.conf"
    apache_vhosts_template: "vhosts.conf.j2"

If set to true, a vhosts file, managed by this role's variables (see below), will be created and placed in the Apache configuration folder. If set to false, you can place your own vhosts file into Apache's configuration folder and skip the convenient (but more basic) one added by this role. You can also override the template used and set a path to your own template, if you need to further customize the layout of your VirtualHosts.

    apache_global_vhost_settings: |
      DirectoryIndex index.php index.html
      # Add other global settings on subsequent lines.

You can add or override global Apache configuration settings in the role-provided vhosts file (assuming `apache_create_vhosts` is true) using this variable. By default it only sets the DirectoryIndex configuration.

    apache_vhosts:
      # Additional optional properties: 'serveradmin, serveralias, extra_parameters'.
      - servername: "local.dev"
        documentroot: "/var/www/html"

Add a set of properties per virtualhost, including `servername` (required), `documentroot` (required), `allow_override` (optional: defaults to the value of `apache_allow_override`), `options` (optional: defaults to the value of `apache_options`), `serveradmin` (optional), `serveralias` (optional) and `extra_parameters` (optional: you can add whatever additional configuration lines you'd like in here).

Here's an example using `extra_parameters` to add a RewriteRule to redirect all requests to the `www.` site:

      - servername: "www.local.dev"
        serveralias: "local.dev"
        documentroot: "/var/www/html"
        extra_parameters: |
          RewriteCond %{HTTP_HOST} !^www\. [NC]
          RewriteRule ^(.*)$ http://www.%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

The `|` denotes a multiline scalar block in YAML, so newlines are preserved in the resulting configuration file output.

    apache_vhosts_ssl: []

No SSL vhosts are configured by default, but you can add them using the same pattern as `apache_vhosts`, with a few additional directives, like the following example:

    apache_vhosts_ssl:
      - {
        servername: "local.dev",
        documentroot: "/var/www/html",
        certificate_file: "/home/vagrant/example.crt",
        certificate_key_file: "/home/vagrant/example.key",
        certificate_chain_file: "/path/to/certificate_chain.crt"
      }

Other SSL directives can be managed with other SSL-related role variables.

    apache_ssl_protocol: "All -SSLv2 -SSLv3"
    apache_ssl_cipher_suite: "AES256+EECDH:AES256+EDH"

The SSL protocols and cipher suites that are used/allowed when clients make secure connections to your server. These are secure/sane defaults, but for maximum security, performand, and/or compatibility, you may need to adjust these settings.

    apache_allow_override: "All"
    apache_options: "-Indexes +FollowSymLinks"

The default values for the `AllowOverride` and `Options` directives for the `documentroot` directory of each vhost.  A vhost can overwrite these values by specifying `allow_override` or `options`.

    apache_mods_enabled:
      - rewrite.load
      - ssl.load
    apache_mods_disabled: []

Which Apache mods to enable or disable (these will be symlinked into the appropriate location). See the `mods-available` directory inside the apache configuration directory (`/etc/apache2/mods-available` by default) for all the available mods.

    apache_packages:
      - apache2
      - apache2-utils

The list of packages to be installed. This defaults to a set of platform-specific packages for RedHat or Debian-based systems (see `vars/RedHat.yml` and `vars/Debian.yml` for the default values).

    apache_ignore_missing_ssl_certificate: true

If you would like to only create SSL vhosts when the vhost certificate is present (e.g. when using Let’s Encrypt), set `apache_ignore_missing_ssl_certificate` to `false`. When doing this, you might need to run your playbook more than once so all the vhosts are configured (if another part of the playbook generates the SSL certificates).

## .htaccess-based Basic Authorization

If you require Basic Auth support, you can add it either through a custom template, or by adding `extra_parameters` to a VirtualHost configuration, like so:

    extra_parameters: |
      <Directory "/var/www/password-protected-directory">
        Require valid-user
        AuthType Basic
        AuthName "Please authenticate"
        AuthUserFile /var/www/password-protected-directory/.htpasswd
      </Directory>

To password protect everything within a VirtualHost directive, use the `Location` block instead of `Directory`:

    <Location "/">
      Require valid-user
      ....
    </Location>

You would need to generate/upload your own `.htpasswd` file in your own playbook. There may be other roles that support this functionality in a more integrated way.

Dependencies
------------

None.

## Example Playbook

    - hosts: webservers
      vars_files:
        - vars/main.yml
      roles:
        - { role: sierkje.apache }

*Inside `vars/main.yml`*:

    apache_listen_port: 8080
    apache_vhosts:
      - servername: "example.org"
        documentroot: "/var/www/vhosts/example.org"

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2017 by [Sierk Beij](https://github.com/sierkje).
It is a fork of the [geerlingguy.apache](https://galaxy.ansible.com/geerlingguy/apache) Ansible role by that was created by
[Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
