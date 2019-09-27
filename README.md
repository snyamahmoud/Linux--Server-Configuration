# Linux--Server-Configuration

# FSND Project 3

This is not a copy & paste walkthrough, but it does answer some questions
you might have found out by research on your own. Use with caution.

Reviewers, please see /home/grader/README on the VM for details not included 
here. The application is running on:

  http://ec2-54-68-6-64.us-west-2.compute.amazonaws.com/
  IP address: 54.68.6.64
  SSH port: 22
  
Addendum: the server part of the README is now available in this repository as 
README_server_part.md. If you are still working on this project, this file is
full of spoilers!

## 1. The VM

Udacity will provide a virtual machine running in the Amazon cloud. The
IP address is given on your Udacity account page. The VM can be also be
managed (as in: created/started and deleted) from that page.

You will not find the page "Development Environment" in the "My Account"
menu on the left. The link is given in the project details for P5.

## 2. SSH

On your hidden development environment page you can also download the 
private ssh key to use for login as root to your Amazon instance. To use it,
download the private key, and move it to the .ssh directory in your home.
Make sure the file has the correct permissions, or ssh will refuse to
use it.

**Google terms: 'ssh private key location', 'ssh permissions too open'**
(These steps are also documented on the development environment page.)

## 3. Users

You will be asked later to disable direct logins for the root user for 
security reasons. If you do so, you need a different user to login with.
In this project, it is specified that this user should be called 'grader'.

Create this as a standard Linux user.

**Google terms: 'ubuntu add user'**

(If you are bored, find out the difference between the 'adduser' and 
'useradd' commands.)

## 4. Sudo

There are several ways to allow a user to execute commands with sudo. One
simple way prepared for you by the Debian/Ubuntu distribution maintainers
is using the /etc/sudoers.d/ directory. For details:

**Google terms: 'ubuntu sudoers.d'**

In this step, make sure the 'grader' user can use sudo.

### 4.1. SSH public key login for user 'grader'

This is only listed in the rubric, not in the project details.

You will be asked to disable root login by only allowing ssh login to 
non-root users. Also, ssh login should only be possible with ssh public 
keys, password authentication should be disabled. 

This is a very common security best practice: attackers know that every 
system has a user 'root' and will try to login with that account. Attackers 
do not know how you named your non-root users. If password login is enabled 
and no further measures taken, an attacker can try to guess the password 
indefinitely and may eventually get access that way.

**Google terms: 'ssh key authentication'**

You can either create a new key pair for the 'grader' user, or just use the
private key that has already been created and configured for the root user.

You will be asked to hand over the private key for that key pair when you
submit the project, so don't use a key pair that you actually use in real 
life.

Take care that the file permissions on all ssh-related files are set 
correctly (ssh will complain). While you are playing around with SSH keys 
(and later, configuration), make sure you ALWAYS stay logged in in at least 
one shell window. Do NOT log out until you have successfully logged in with 
the new configuration/user/key, or you might lock yourself out of the 
system.

## 5. Update all installed packages

**Google terms 'ubuntu update packages'**

Manual update is easy -- don't forget you need use sudo to do this, and
don't let the different subcommands of apt-get confuse you.

In the rubric you'll find that for 'exceeds specifications' you need to
configure automatic unattended package updates. This is a bit controversial,
as every update also has a chance to break your system due to configuration
or setup changes you need to implement in your applications to run them on 
the newer versions of your platform (web server, databases etc.).

It is, however, possible to only install security updates automatically. 
This might be a good compromise depending on your needs. Security updates 
should be installed as soon as possible, and the distributions (usually) 
don't include dangerous changes or version upgrades in security updates. You   still should subscribe to the security announcement channels of the 
distribution you are using, and of course monitor the activity of any 
automatic updates you have set up, e.g. by having your scripts or tools
send you notification emails.

**Google terms 'ubuntu automatic security updates'**

## 6. Use a non-standard port for SSH

The reason for this is similar to that for not using the root account
for login: attackers know that the ssh service is usually listening on port
22, so they point their attacks there. If ssh is listening elsewhere, they
at least have to find out where first.

**Google terms: 'linux sshd use different port'**

The project asks us to use port 2200. As said above, don't log out of
your working ssh session before you have confirmed you can log in on the
new port!

Also, be sure you understand the difference between 'ssh' and 'sshd' and
their respective config files.

### 6.1. Disable root ssh login

This is only listed in the rubric, not in project details: as mentioned
above, decline any login for the root account through ssh, always. This
is done in the ssh configuration, look at the corresponding config file
for hints, or **google 'linux ssh disable root login'**.

## 7. Firewall

You can set up your system in a way that it only accepts traffic on
certain ports and blocks everything else. The traditional linux tool to
do that is 'iptables'. As this is powerful but hard to use for beginners,
ubuntu comes with a firewall setup tool that is much easier to use: 'ufw',
the 'uncomplicated firewall'.

**Google terms: 'linux firewall setup', 'ubuntu ufw'**

The project explicitly asks us to use 'ufw', so only spend time on 
'iptables' documentation if you are curious. It is not needed to configure
the firewall as requested.

We are asked to block everything, and allow incoming connections for these
services:

- SSH on port 2200 (we moved it there in the step above)
- HTTP on port 80 (we will run a webserver later)
- NTP on port 123 (network time protocol, explained in the next step)

The ubuntu documentation of UFW is detailed enough to do this. You configure
the firewall by calling

>$ sudo ufw [allow|deny] [service|port]

Don't forget to enable the firewall when you're finished configuring it. 
Also, when it is enabled, make sure you can still log in before logging 
out of your running ssh session! This is another excellent opportunity to
lock yourself out from your own VM.

If you are curious where the configuration you entered on the command line
is stored, **google 'ubuntu ufw rules stored'**. Also remember that the
'ufw status' command has a 'verbose' option for more information.

I was wondering why we are asked to open port 123 for ntp: the firewall
in general allows all connections initiated by a service on the system
(i.e. 'outgoing'), and ntp runs as a daemon and should be covered by this.

However, I found a few discussions of this questions when **googling for
'ntp firewall bidirectional'**, and as it's explicitly written in the
project details, I've opened the port as well.
  
### 7.1. Monitoring

For 'exceeds expectations' we are asked to monitor unsuccessful connection
attempts, block the IP addresses they come from, and send an email
notification about the fact.

A good starting point is **googling for 'ubuntu block ip after failed 
login'**.

When googling, remember that we are using 'ufw' for firewall configuration
instead of 'iptables', you might want to refine your search terms.

## 8. Date and Time

The project details ask for the system to be running on UTC time.

**Google terms: 'timezone utc gmt'** for a bit of (historical) background
about timezones and UTC,

**Google terms: 'ubuntu set timezone'** for how to do it.

NTP is a service that requests accurate time from external time servers
and adjusts the server clock accordingly. It is not specified anywhere that 
ntp should be set up, but a) you really want the correct time for all sorts
of reasons (**google 'why ntp is important'**, and b) as we already opened 
the ntp port in the firewall we can just as well use it.

**Google terms: 'ubuntu ntp'**

There are several options, I chose the one where a daemon is always running
and taking care of the time synchronisation. 

## 9. Apache Web Server & WSGI

Apache is a web server implementing the HTTP standard. On its own it
delivers files from the file system to the user's browser. If there is
code to be executed, i.e. a full-fledged web application backend running,
Apache is usually extended with modules.

WSGI is another standard that defines how web application backends written
in Python can be served by web servers.

**Google terms: 'http how does it work', 'apache http', 'ubuntu apache',
'apache install modules', 'apache wsgi'**

If you haven't installed a web server before, don't despair and take the
time to do a few of the simpler tutorials, set up a static web site and set
up a 'Hello World' app with WSGI first.

## 10. PostgreSQL

The database to use in this project is PostgresQL, and it is included
in the Ubuntu distribution and can be installed with apt-get. You will
need both the server and the client packages.

**Google terms: 'ubuntu postgresql'**

### 10.1. Disable remote connections

The web application will run on the same system as postgresql, and it will
be the only application that needs access to the database. Therefore there
is no need for remote access to the database, and for security reasons it
is best to disable it completely (if it isn't disabled by default).

**Google terms: 'postgresql remote access'**

Also note that the firewall would have blocked access to the postgresql
port anyway, as we haven't explicitly opened it in the firewall
configuration step.

**Google terms: 'postgresql ports'** 

In this step it is helpful to also find out how to connect to a postgres
database with the command line client, and to look around what the basic
setup looks like, what users already exist etc.

### 10.2. Create database user and database for the application

The application obviously needs access to the database. To make sure that
even a potentially successful attacker can do as little harm as possible,
it is common practice to create a separate database user for each 
application, and to give it only the privileges it really needs. As the app 
uses SQLalchemy to manage the database, the database user must be able to 
create and drop tables, but it is not necessary that it can create or drop 
the database itself.

There is a convention how to name database users and their 'default'
databases, it makes sense to follow it here.

The project details ask for the database user to be named 'catalog', I
chose to give it the same name as my application ('bookswapping').

**Google terms: 'ubuntu postgresql steup', 'postgres create user', 
'postgres create database', 'postgres user privileges', 'postgres grant 
on database'**

If you create the database for the application and the corresponding
database user and still can't login through psql, googling for the
exact error message will most likely help.

Postgresql has different authentication methods. One of them, 'peer',
assumes that there is a system user with the same name as the database
user, and uses system calls to linux to authenticate that user. If you
choose NOT to create a system user matching your database user, this
mechanism fails.

It also makes a difference whether you call psql with the parameter '-h
127.0.0.1', telling it to connect to localhost through a network connection,
or without any parameter specifying the kind of connection. In the latter
case, it defaults to using unix domain sockets (**google** that for more
information).

For more details, and in case of login problems, find and read the file 
'pg_hba.conf'. At the end of this step you should be able to log in with
your newly created database user to postgresql, and this user should be
able to create and drop tables. This can be tested with psql.


## 11. Deploy app

### 11.1. Install git and checkout app

Git is available through apt-get, just install it.

Before you can checkout the application, you need to choose the directory it
will be run from.

By convention, files and applications that are served by a web server
are stored below /var/www on Ubuntu (and Debian). The default document
root for apache is /var/www/html.

**Google terms: 'apache docroot'**

The document root (often shortened to 'docroot') is necessary when
files are served directly from the file system. Everything below the
docroot is accessible to the world, unless further restrictions are
configured. Everything outside this directory hierarchy is invisible to
the web server.

Our python web application, however, will not be served directly from the
file system, but through the WSGI module. I therefore decided to keep it
in a directory outside the docroot. I chose '/var/www/bookswapping' as
base directory. This directory will contain the git checkout in
'/var/www/bookswapping/bookswapping' and the file 
'/var/www/bookswapping/app.wsgi' to configure and start the application.

I chose to checkout the application code as user 'grader', this user
is also the owner of the directory '/var/www/bookswapping'. The web
server will need to be able to read from, but not write to this directory.

### 11.2. Install necessary python dependencies

The application needs external python libraries, and these can be installed
in the default system python directories, so that they are accessible by 
default by all users and applications, or -- elsewhere. I chose 'elsewhere',
because there can be more than one python application running on a system,
and they might need different versions of certain libraries. Having the
dependencies of each application in separate directories keeps things
clean and easy to manage.

I chose the directory '/var/www/bookswapping/python' for all python
dependencies of my application. 'pip' can be told to use a different
directory for installation. It also can be told to not rely on python
libraries that are already present on the system, but to really install
everything into that target directory.

**Google terms: 'pip install to specific directory'**

Note that no matter where you choose to install the additional libraries,
pip will download the sources and compile them, so depending on your project 
you will need to install additional packages containing development 
libraries and tools. 

### 11.3. Use the local postgres database

Find the code in your application where you connect to the database, and
change it to use the newly created local postgres database.

**Google terms: 'sqlalchemy postgres database url format'**

You will need to enter the database password there. This is one of the 
reasons why you should make sure that apache does not serve your python 
files as plain text (and why I do not use a directory below the docroot).

### 11.4. Make the application known to apache

My main WSGI entry point is the file app.wsgi. In my case, the wsgi file is
not in the same directory as the rest of the application, so I had to make
sure that the application directory is in python's search path.

**Google terms: 'flask wsgi'**

To enable WSGI, it needs to be configured in the apache configuration files
below /etc/apache2. As I don't have any other sites or applications running
in this VM, I chose /etc/apache2/sites-enabled/000-default.conf to put
all my new apache directives in.

**Google terms: 'apache wsgi'**

As I have installed the python libraries my project needs into a custom
path, I had to let the WSGI module know about this path. Also, my application 
contains static files in the 'static' directory, so I had to make this
directory available under '/static' as well.
 
(How the apache configuration files are organized below /etc varies between 
linux distributions. For details on the Debian/Ubuntu way, **google 'debian 
apache config'**.)

### 11.5. Adjust the application to running as WSGI app

My application reads in some information from json files in the application
directory. When the app is running in WSGI, the working directory is no
longer the directory where the .py files are stored so the json files could
no longer be found.

After **googling for 'wsgi current working directory'** I had to make a
small code change in my application to fix that problem.

At the end of this step, when you (re)start apache, your application should
be running and accessible on port 80 on your VM.

### 11.6. Check security settings: .git directory, .py files, /static

The only directory below /var/www/bookswapping/bookswapping that is
directly served by apache is 'static'. I don't want people to look at the
content of directories, so I have forbidden directory listings for this
directory.

**Google terms: 'apache disable directory listing'**

All other requests go through the Flask application, which responds with
'404 - File not found' HTTP errors on all paths which are not configured
as routes within the app.

To make sure that no content is served that shouldn't be, I test with the
following URLs:

- http://MYURL/.git/
- http://MYURL/static/
- http://MYURL/database.py
- http://MYURL/static/../database.py

### 11.7. Make the new location known to OAuth providers 

Udacity only gives us an IP address to connect to. To register the application
at the OAuth providers used for login, I need a fully-qualified domain name.

This can be found out through **reverse DNS lookup** services (google that!).

Once the new location has been registered, OAuth login should be working.

## Addendum I: sending / delivering local email

Some tasks ask for notification emails being sent in certain situations.

For this to work, you need a local MTA (mail transport agent) to be installed
that will accept and deliver the emails. The most popular alternatives on
linux systems are sendmail, exim4 and postfix, of which I _think_ postfix is
easiest to configure. The ubuntu package even contains a few default 
configurations, of which 'local-only' is sufficient for this project.

The choice of MTA is subjective, of course.

If you want to learn more about email systems and the abbreviations used
to describe them, **google 'mta mua'**. ('MUA' is the mail user agent, the 
mail client an end user uses for handling email.)

If you choose postfix, googling 'ubuntu postfix' will point you at the
official ubuntu manual section.

If you want to test your local mail setup, you can use the command 'mail'
from the package 'bsd-mailx' to send an email from the command line.

**Google terms 'linux send email command line'**

Make sure to read the help articles you'll find to the end, the usage
of 'mail' is not really intuitive, but it is a very helpful tool.

(An alternative provider of the 'mail' command is the package GNU mailutils,
but it will install more dependent packages, which is why I prefer 
bsd-mailx'.

## Addendum II: Reading local email

For reading email on the command line, I prefer 'mutt', a very powerful
text-based email client (MUA). If you already know the 'nano' text editor,
you might want to check out 'pine' instead which has a similar user 
interface.

If you don't want to install a MUA, you can always read your email with
less:

**$ less /var/mail/grader**

and **google 'linux mail spool'** for some background why it ends up there.
 
## Addendum III: Monitoring system health 

I want to setup Nagios to monitor that Apache and PostgreSQL are running.
If either Apache or PostgreSQL are not running, I want Nagios to send out
an email notification to grader@localhost.

**Google terms: 'ubuntu nagios'**

On Ubuntu, Nagios 3 is available through apt-get. This is not the newest
version, but I don't want to compile nagios from source code, so I use
that version.

Nagios is often used in a distributed manner, where one central server
collects monitoring information from several nodes. This setup is much
simpler, as I only have one VM to monitor and run nagios itself on.

The check for a running HTTP server is included in the nagios3 sample
configuration. I added a check for PostgreSQL. For this to work, I allowed
the nagios user to connect to the PostgreSQL database without a password
in pg_hba.conf
