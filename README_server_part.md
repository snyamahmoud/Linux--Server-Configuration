Dear reviewer,

for my detailed notes for this project. Additional information:

1. My VM is running on

  http://ec2-54-68-6-64.us-west-2.compute.amazonaws.com/

and its IP address is `54.68.6.64`.

All email notifications go to `grader@localhost`. The text-based MUA `mutt`
is installed.

2.-4. Please log in as user `grader` with the private ssh key provided. SSH
is running on port 2200. You may use sudo without a password whenever you
need superuser access.

5. I have configured automatic installation of security updates with the
package 'unattended-upgrades'. It will send an email to `grader@localhost` when
it installed something.

I have also installed and configured apticron to send an email to
`grader@localhost` whenever updates for installed packages are available.

6. SSH is listening on port 2200
File: `/etc/ssh/sshd_config`
Relevant line(s):
  Port 2200

Root login with SSH is disabled, password login is disabled for all users
File: `/etc/ssh/sshd_config`
Relevant line(s):
  PermitRootLogin no
  PasswordAuthentication no

7. UFW has been set up according to specification, please call

  $ sudo ufw status verbose

to verify.

I have installed and configured fail2ban to dynamically add a firewall
rule to temporarily block an IP address after 6 unsuccessful login attempts.

It will also send a notification email to grader@localhost.

Files:
`/etc/fail2ban/jail.local`
`/etc/fail2ban/action.d/ufw.conf`

To test this, try to login as any non-existing user 7 times in a row from
a different IP address that the one you are now connected from. Observe that
the last connection attempt will time out. Check grader's email for the
notification, and

  $ sudo ufw status verbose

for the newly inserted firewall rule.

8. I have installed and configured ntpd to always keep the system time in sync
with public time servers.

I have used

  $ sudo dpkg-reconfigure tzdata

to set the system's timezone to UTC.

9. I have installed apache2 and mod_wsgi with all dependencies.

I then used the documentation at:

  https://code.google.com/p/modwsgi/wiki/QuickConfigurationGuide

to install a 'Hello World' application served through mod_wsgi to verify
that it works.

10. I have installed postgresql and postgresql-client with all dependencies.

The basic installation of postgres on Ubuntu does not allow remote connections.

I verified this by checking the files:

File:
`/etc/postgresql/9.3/main/postgresql.conf`
Relevant lines:
  #listen_addresses = 'localhost'         # what IP address(es) to listen on;
                                          # comma-separated list of addresses;
                                          # defaults to 'localhost'; use '*' for all

File:
`/etc/postgresql/9.3/main/pg_hba.conf`
Relevant lines (actually irrelevant as postgres is only listening on localhost
anyway, see above. The connections specified here still only come from
localhost / through unix domain sockets, so no remote connections):
  # "local" is for Unix domain socket connections only
  local   all             all                                     peer
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  # IPv6 local connections:
  host    all             all             ::1/128                 md5

In addition, the port posgreslq is running on (5432, see `/etc/postgresql/9.3/main/postgresql.conf`) has not been opened in ufw.

I have then created the database user 'bookswapping' and the postgres database
'bookswapping'.

I've granted the user 'bookswapping' all privileges on the database
'bookswapping'.

I did not need to adjust /etc/postgresql/9.3/main/pg_hba.conf, the user
can connect to the database on localhost and log in with its password.

I did _not_ create a system user 'bookswapping'. All application files
are owned by the user 'grader'.

11. I've created the following files and directories for my application:

- /var/www/bookswapping/app.wsgi -- WSGI wrapper file
- /var/www/bookswapping/python -- python libraries needed to run the application
- /var/www/bookswapping/bookswapping -- git checkout of the application

Please see my public notes at https://github.com/skh/fsnd-linux-server-config
for the reasoning behind these choices.

I have used pip with the options --target and --ignore-installed to compile
and install all necessary dependencies into /var/www/bookswapping/python. I
had to install postgresql-server-dev-9.3 and libpython-dev (build-essential
was already installed).

I have changed the DATABASE_URL in the application code (database.py) to
point to the local postgres database.

I also had to do this change:

  https://github.com/skh/bookswapping/commit/01edcc

to be able to read settings files from the file system. This was necessary
because the current working directory of a python app run through WSGI is not
necessarily the one where the python files are stored, so relative paths are
no longer working.

I have followed

- http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/

to create the WSGI wrapper file.

The apache/mod_wsgi configuration can be found in

- /etc/apache2/sites-enabled/000-default.conf

I have used

- http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/ and
- https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives

for the apache/mod_wsgi configuration.

I have then registered the new URL of my application with the OAuth providers
I'm using:

- Facebook: https://developers.facebook.com/
- Google+: https://console.developers.google.com/

(I have used http://ping.eu/rev-lookup/ to find out the fully qualified
domain name of my VM, as I've been given only the IP address.)

Addendum: Monitoring

I have installed and configured Nagios 3 to monitor whether Apache and
PostgreSQL are running. Nagios sends email to grader@localhost if it
detects problems with one of these services. The Nagios frontend is
available at:

  http://ec2-54-68-6-64.us-west-2.compute.amazonaws.com/nagios3/
  user: nagiosadmin
  password: N0g4hj$2

(On a production server this information would not be hosted on a server
visible to the world, and password-protected pages would only be accessible
through SSL on port 443. As it is now, the password is transmitted
unencrypted.)

The configuration of Nagios 3 is very close to the default configuration that
comes with the Ubuntu packages. I have added the hostgroup 'pg-servers' in
/etc/nagios3/conf.d/hostgroups_nagios2.cfg and the service check for
PostgreSQL in /etc/nagios3/conf.d/services_nagios2.cfg. I have also commented
out the parts of the default configuration that I don't need.

For the service check for PostgreSQL I used the check_pgsql command that
comes with nagios. It connects to the template1 database and immediately
disconnects again. For this to work, I had to create the user 'nagios'
in postgresql and enable passwordless login for it from localhost with these
lines in /etc/postgresql/9.3/main/pg_hba.conf:

# Allow nagios user from localhost to connect without a password
host    template1       nagios          127.0.0.1/32            trust

The configuration of contacts which receive email notifications is contained
in /etc/nagios3/conf.d/contacts_nagios2.cfg, I have added the user 'grader'
there.
