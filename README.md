# Ansible Role: Certbot (for Let's Encrypt)

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-certbot.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-certbot)

Installs and configures Certbot (for Let's Encrypt).

## Requirements

Certbot requires Git to be installed if one wants to install Certbot from Git repository instead of package management. You can install Git using the `geerlingguy.git` role.

## Role Variables

    certbot_from_git: no
    certbot_repo: https://github.com/certbot/certbot.git
    certbot_version: master
    certbot_keep_updated: yes

Certbot Git repository options. This role clones the agent from the configured repo, then makes the `certbot-auto` script executable if `certbot_from_git` is `yes`. Otherwise it will be installed from distribution's package management.

    certbot_dir: /opt/certbot

The directory inside which Certbot will be cloned when using Git.

    certbot_auto_renew: true
    certbot_auto_renew_user: "{{ ansible_user }}"
    certbot_auto_renew_hour: 3
    certbot_auto_renew_minute: 30

By default, this role configures a cron job to run under the provided user account at the given hour and minute, every day. The defaults run `certbot renew` (or `certbot-auto renew`) via cron every day at 03:30:00 by the user you use in your Ansible playbook. It's preferred that you set a custom user/hour/minute so the renewal is during a low-traffic period and done by a non-root user account.

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

After installation, you can create certificates using the `certbot` (or `certbot-auto`) script, which by default is installed inside the configured `certbot_dir` (when using Git). Here are some example commands to configure certificates with Certbot:

    # Automatically add certs for all Apache virtualhosts (use with caution!).
    /opt/certbot/certbot-auto --apache

    # Generate certs, but don't modify Apache configuration (safer).
    /opt/certbot/certbot-auto --apache certonly

By default, this role adds a cron job that will renew all installed certificates once per day at the hour and minute of your choosing.

You can test the auto-renewal (without actually renewing the cert) with the command:

    /opt/certbot/certbot-auto renew --dry-run

See full documentation and options on the [Certbot website](https://certbot.eff.org/).

## License

MIT / BSD

## Author Information

This role was created in 2016 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
