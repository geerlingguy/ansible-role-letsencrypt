# Ansible Role: Certbot (for Let's Encrypt)

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-certbot.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-certbot)

Installs and configures Certbot (for Let's Encrypt).

## Requirements

If installing from source, Git is required. You can install Git using the `geerlingguy.git` role.

## Role Variables

The variable `certbot_install_from_source` controls whether to install Certbot from Git or package management. The latter is the default, so the variable defaults to `no`.

    certbot_auto_renew: true
    certbot_auto_renew_user: "{{ ansible_user }}"
    certbot_auto_renew_hour: 3
    certbot_auto_renew_minute: 30

By default, this role configures a cron job to run under the provided user account at the given hour and minute, every day. The defaults run `certbot renew` (or `certbot-auto renew`) via cron every day at 03:30:00 by the user you use in your Ansible playbook. It's preferred that you set a custom user/hour/minute so the renewal is during a low-traffic period and done by a non-root user account.

### Certificate generation
`certbot_handle_certs: true`
Set to true to have this role register and generate certificates for your
domains.

`certbot_register_email: ''``
The email to register with. This is required (if you set `certbot_handle_certs` to true) or else the role will fail.

certbot_domains: []
The domains to generate certs for. This is required (if you set `certbot_handle_certs` to true) or else the role will fail. If you are also using the [geerlingguy/ansible-role-apache](https://github.com/geerlingguy/ansible-role-apache) role, you can use something like this to get a list of your domains:
```yaml
certbot_domains: "{{ [0,1,2] | map('extract', apache_vhosts, 'servername') | list }}"
```
where `[0,1,2,]` means the first, second and third virtual host specification respectively.

`certbot_register_command: "{{ certbot_script }} register --non-interactive --agree-tos --email {{ certbot_register_email }}"`
The command to run to register with Let's Encrypt.

`certbot_cert_command: "{{ certbot_script }} certonly --noninteractive --standalone"`
The command to run to generate the certificates.

### Source Installation from Git

You can install Certbot from it's Git source repository if desired. This might be useful in several cases, but especially when older distributions don't have Certbot packages available (e.g. CentOS < 7, Ubuntu < 16.10 and Debian < 8).

    certbot_install_from_source: no
    certbot_repo: https://github.com/certbot/certbot.git
    certbot_version: master
    certbot_keep_updated: yes

Certbot Git repository options. To install from source, set `certbot_install_from_source` to `yes`. This clones the configured `certbot_repo`, respecting the `certbot_version` setting. If `certbot_keep_updated` is set to `yes`, the repository is updated every time this role runs.

    certbot_dir: /opt/certbot

The directory inside which Certbot will be cloned.

## Dependencies

None.

## Example Playbook

    - hosts: servers

      vars:
        certbot_auto_renew_user: your_username_here
        certbot_auto_renew_minute: 20
        certbot_auto_renew_hour: 5

      roles:
        - geerlingguy.certbot

### Manually creating certificates with certbot

If you do not enable certificate generation via this role (`certbot_handle_certs: true`), you can follow the steps below to generate your certificates.

After installation, you can create certificates using the `certbot` (or `certbot-auto`) script (use `letsencrypt` on Ubuntu 16.04, or use `/opt/certbot/certbot-auto` if installing from source/Git. Here are some example commands to configure certificates with Certbot:

    # Automatically add certs for all Apache virtualhosts (use with caution!).
    certbot --apache

    # Generate certs, but don't modify Apache configuration (safer).
    certbot --apache certonly

If you want to fully automate the process of adding a new certificate, you can do so using the command line options to register, accept the terms of service, and then generate a cert using the standalone server:

  1. Make sure any services listening on port 80 (Apache, Nginx, Varnish, etc.) are stopped.
  2. Register with something like `certbot register --agree-tos --email [your-email@example.com]`
    - Note: You won't need to do this step in the future, when generating additional certs on the same server.
  3. Generate a cert for a domain whose DNS points to this server: `certbot certonly --noninteractive --standalone -d example.com -d www.example.com`
  4. Re-start whatever was listening on port 80 before.
  5. Update your webserver's virtualhost TLS configuration to point at the new certificate (`fullchain.pem`) and private key (`privkey.pem`) Certbot just generated for the domain you passed in the `certbot` command.
  6. Restart your webserver so it uses the new HTTPS virtualhost configuration.

### Certbot certificate auto-renewal

By default, this role adds a cron job that will renew all installed certificates once per day at the hour and minute of your choosing.

You can test the auto-renewal (without actually renewing the cert) with the command:

    /opt/certbot/certbot-auto renew --dry-run

See full documentation and options on the [Certbot website](https://certbot.eff.org/).

## License

MIT / BSD

## Author Information

This role was created in 2016 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
