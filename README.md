## Drush Queue Checker ##

This is a Nagios Plugin script meant for monitoring a drush queue meant for invalidating data.  This was originally written due to the constant breakage of the purge module in Drupal and the ever accumulating queue that may go unnoticed for hours to weeks depending on site maintenance schedules.

This script covers the use of site and server integrated drush command and does not include any usage of  

### Usage ###

```
usage: check_drush_queue [-vV] -p <drupal root> -d <drush path> -w <value> -c <value>
     -v             	    - print out version information
     --version
     -V                     - print verbose information
     --verbose
     -p <drupal root>	    - path to the root of the Drupal site
     -d <drush path> 	    - path to drush (absolute, or relative to the Drupal root)
     -w <warn count>        - drush queue size WARNING value
     -c <critical count>    - drush queue size CRITICAL value
```

### Setup ###

If using a server, copy your check_drush_queue script up to the server and place where appropriate.  That may be the
nagios plugin directory in `/usr/lib64/nagios/plugins/` or another location of your choosing. 

#### Using NRPE ####

Setup a command within your nrpe.cfg file (usually `/etc/nagios/nrpe.cfg`) on the server the Drupal site exists on which Nagios can poll:

example:
```
command[check_drush_queue]=/usr/lib64/nagios/plugins/check_drush_queue -p /var/www/example.com/current/ -d vendor/bin/drush -w 1000 -c 3000
```

#### Using NCPA ####

No documentation currently.  Feel free to contribute!


### Troubleshooting ###

Should you have your site directories unreadble by users other than those that are owners or group members, you can do one of two things.

1. Add the nrpe user to the group able to read the directories and have the ability to execute drush

2. sudo your check command:
```
command[check_drush_queue]=sudo /usr/lib64/nagios/plugins/check_drush_queue -p /var/www/example.com/current/ -d vendor/bin/drush -w 1000 -c 3000
```

Additionally you will want to create a file in `/etc/sudoers.d/` to specify exactly the nrpe user is able to do:
example: `/etc/sudoers.d/10-nagios`

```
Defaults:nrpe !requiretty
nrpe	ALL=(ALL)	NOPASSWD:	/usr/lib64/nagios/plugins/check_drush_queue
```

If your system is also running SELinux (eg. RedHat EL 7/8/9 or a distribution based off of it) your commands will also likely be blocked in two different spots.
The first requires the correct SELinux permissions on the `check_drush_queue` script.
```
chcon -u system_u -t nagios_unconfined_plugin_exec_t /usr/lib64/nagios/plugins/check_drush_queue
```

Then to have SELinux allow the NRPE user to be able to run sudo commands one will have to tell SELinux to do so.
```
setsebool -P nagios_run_sudo 1
```
This tells SELinux that nagios(nrpe) is allowed to run sudo commands.
