# Webarchitects OpenProject Ansible Role

Currently this role only [installs OpenProject](https://www.openproject.org/docs/installation-and-operations/installation/), configuration needs to be done using the wizard:

```bash
sudo openproject configure
```

Manually [configure outgoing email](https://www.openproject.org/docs/installation-and-operations/configuration/outbound-emails/) so that a password reset can be sent for the admin account:

```bash
openproject config:set OPENPROJECT_EMAIL__DELIVERY__METHOD=smtp
openproject config:set OPENPROJECT_MAIL__FROM=openproject@$(hostname -f)
openproject config:set OPENPROJECT_SMTP__ADDRESS=localhost
openproject config:set OPENPROJECT_SMTP__PORT=25
openproject config:set OPENPROJECT_SMTP__DOMAIN=$(hostname -f)
openproject config:set OPENPROJECT_SMTP__AUTHENTICATION=none
openproject config:set OPENPROJECT_SMTP__ENABLE__STARTTLS__AUTO=false
openproject config:set OPENPROJECT_SMTP__OPENSSL__VERIFY__MODE=none
openproject restart
```

Get [all the environment variables](https://www.openproject.org/docs/installation-and-operations/configuration/environment/#supported-environment-variables):

```bash
openproject run bundle exec rake setting:available_envs
```

See the [upgrade instructions](https://www.openproject.org/docs/installation-and-operations/operation/upgrading/) and version [release notes](https://www.openproject.org/docs/release-notes/).

## Notes

* The upgrade to [OpenProject 13.4.0](https://www.openproject.org/docs/release-notes/13-4-0/) required the following to be run:

```bash
sudo -i
su - postgresql
psql openproject
GRANT CREATE ON SCHEMA public TO openproject;
\q
exit
exit
```

* The default Apache config doesn't work with Let's Encrypt and the `Includes` for non existant files causes a problem, the following config with the [users role](https://git.coop/webarch/users) replicates the Apache config provided by OpenProject:

```yaml
users:
  openproject:
    users_apache_virtual_hosts:
      openproject:
        users_apache_server_name: "{{ inventory_hostname }}"
        users_apache_vhost_docroot: false
        users_apache_expires: medium
        users_apache_directories:
          "/opt/openproject/public":
            users_apache_override:
              - "None"
            users_apache_options:
              - "-Indexes"
        users_apache_headers:
          - type: request
            action: setifempty
            arg: X-Forwarded-Proto https
        users_apache_alias:
          - url: /assets
            path: /opt/openproject/public/assets
          - url: /uploads
            path: /opt/openproject/public/uploads
        users_apache_locations:
          - location: "/"
            proxy_pass: http://127.0.0.1:6000/
            reverse: true
          - location: "/assets"
            proxy_pass: "!"
          - match: "^/sys"
            authname: Local connections only
            authtype: None
            require:
              - local
    users_editor: vim
    users_home_mode: "0755"
    users_shell: /bin/bash
    users_skel: /usr/local/etc/skel.d/www
    users_state: present
    users_system: true
```

## TODO

* The `/etc/openproject/installer.dat` file could be created by this role and `openproject configure` could be run, see [#1](https://git.coop/webarch/openproject/-/issues/1).

## Repository

The primary URL of this repo is [`https://git.coop/webarch/openproject`](https://git.coop/webarch/openproject) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-openproject) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/openproject).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/openproject/-/releases).

## Copyright

Copyright 2023-2024 Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
