# Project 5 for Udacity Full Stack Web Developer Nanodegree (FSND)

This project consisted of:
- Securing and configuring a barebones Linux web server
- Deploying the catalog app from project 3, using the Apache web server, Flask Python framework, and PostgreSQL

Server Info:
- Server IP Address: 52.32.90.150
- SSH Port:  2200
- URL to Catalog App: http://ec2-52-32-90-150.us-west-2.compute.amazonaws.com

## Software/Packages Installed
- finger
- Apache2
- mod_wsgi
- pip
- virtualenv
- Flask
- PostgreSQL
- python-psycopg2
- libpq-dev
- Flask-SQLAlchemy
- flask-seasurf
- git
- httplib2
- google-api-python-client
- fail2ban
- iptables-persistent
- sendmail
- mailutils
- php5
- nagios
- [check_postgres](https://bucardo.org/wiki/Check_postgres) - a Nagios plugin for PostgreSQL

## Configuration Changes Made

### Basic Configuration

1. Created user "grader".
2. Gave user "grader" sudo privileges by adding `grader` file to `/etc/sudoers.d` with the following line: `grader ALL=(ALL) NOPASSWD:ALL`
3. Enabled SSH login for "grader" user:
    - Copied `/root/.ssh/authorized_keys` to `/home/grader/.ssh/authorized_keys`.
    - Changed permissions for `/home/grader/.ssh` to 700.
    - Changed permissions for `/home/grader/.ssh/authorized_keys` to 644.
    - Change owner and group for `/home/grader/.ssh` and `/home/grader/.ssh/authorized_keys` to `student` and `student`.
4. Disabled remote root login by editing `/etc/ssh/sshd_config`: `PermitRootLogin no`.
5. Verified that password authentication is disabled by checking `/etc/ssh/sshd_config` for `PasswordAuthentication no`.
6. Updated package source lists: `sudo apt-get update`
7. Updated installed packages: `sudo apt-get upgrade`
8. Set timezone to UTC using `sudo dpkg-reconfigure tzdata`.

#### Automated Security Updates
- Configured weekly cron script `/etc/cron.weekly/apt-security-updates` to install security updates using `aptitude safe-upgrade`, which only installs updates if the updates do not require adding or removing any dependencies.
- The script generates a log in `/var/log/apt-security-updates`. Log rotation is handled by `logrotate.d`.

### Security Configuration

1. Changed ssh default port to 2200 by editing `/etc/ssh/sshd_config`.
2. Configured the Uncomplicated Firewall for ssh, http, and ntp:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow http
sudo ufw allow 123
sudo ufw enable
```

#### Monitoring SSH Login Attempts
- Using Fail2ban to monitor login attempts on SSH and ban ip addresses with repeated failed login attempts.
- To see Fail2ban in action:
    1. Edit `/etc/ssh/sshd_config` to allow password-based logins.
    2. Edit `/etc/fail2ban/jail.local` and change `destemail` to your email address.
    3. Stop and start the fail2ban service:
        - `sudo service fail2ban stop`
        - `sudo service fail2ban start`
    4. **From a remote server**: attempt to ssh in with something like `ssh blah@52.32.90.150 -p 2200`. Type something random in when asked for a password. Repeat this until you stop seeing "Permission denied" messages. This indicates Fail2ban has now blocked the ip.
    5. Check your email. You should receive a notification about the ban.
    6. You can also run `sudo iptables -S` (from your "grader" user login) to see your banned ip address added to the rules.
    7. Please put everything back as you found it.

### Apache Configuration
- Created `CatalogApp.conf` file to tell Apache about my app and placed it in `/etc/apache2/sites-available`.
- Enabled the app using `sudo a2ensite CatalogApp`.
- Changed user and group to `www-data` for app folders within `/var/www/` to correct permissions errors with user-submitted image uploads.
- Removed `Indexes` option from `/etc/apache2/apache2.conf` to remove indexing of `/static` directory.


### PostgreSQL Configuration
1. The default PostgreSQL installation on Ubuntu comes pre-configured to disallow remote connections.
2. Created role `catalog`.
3. Created database `catalog`.
4. Granted all privileges on database `catalog` to user `catalog`.
5. Edited `pg_hba.conf` to include:
`local   catalog     catalog     md5`

### Cloning Git Repo
- Cloned my [Project 3 repo](https://github.com/ppjk1/catalog-app) into `/home/grader/` and copied the directory into `/var/www/CatalogApp/CatalogApp`. In this manner, the .git repo will not be accessible from the web.

### Monitoring Availability
- Using Nagios to monitor HTTP and PostgreSQL availability on localhost and provide emailed alerts.
- To access the web-based dashboard, visit: http://ec2-52-32-90-150.us-west-2.compute.amazonaws.com/nagios. Use the credentials provided in the "Notes to Reviewer".
    - If this were a production server, I would probably lock down access to this dashboard by ip address.
    - Current security hole with the dashboard: password is sent in clear text. Would probably need 'https' to overcome this.
- To see emailed alerts:
    1. Edit `/usr/local/nagios/etc/objects/contacts.cfg` and change `email` to your email address. Save and close.
    2. Reload the Nagios configuration with `sudo service nagios reload`.
    3. Turn off the postgres service: `sudo service postgreql stop`.
    4. Visit the Nagios dashboard and click on `Services`.
    5. After 4 attempts at connecting to Postgres, Nagios will issue a notification and you should see an email arrive in your inbox.
    6. Turn the postgres service back on, and you should (within several minutes) receive another email indicating service is back up.
    7. Please put the configs back as you found them.


### Third-party Resources

- DigitalOcean Tutorial: [Deploying a Flask App on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- DigitalOcean Tutorial: [How to Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
- DigitalOcean Tutorial: [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- Kill the Yak Tutorial: [Use PostgreSQL with Flask or Django](http://killtheyak.com/use-postgresql-with-django-flask/)
- TechArena51 Tutorial: [Python PostgreSQL example](http://techarena51.com/index.php/flask-sqlalchemy-postgresql-tutorial/)
- Udacity Wiki Page: [How to Deploy Your Flask App to Heroku](https://www.udacity.com/wiki/ud330/deploy)
- DigitalOcean Tutorial: [How To Install Git on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)
- DigitalOcean Tutorial: [Configuring ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)
- [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)
- DigitalOcean Tutorial]: [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
- DigitalOcean Tutorial: [How To Install Nagios 4 and Monitor Your Servers on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-nagios-4-and-monitor-your-servers-on-ubuntu-14-04)
