#### ormodernnagios
######Layout and templating
services, hosts,contacts,contact groups,commands,host groups,service groups  
recommend
```
ln -s /usr/local/nagios/etc/ /etc/nagios
```
```
cfg_file=
cfg_dir= /usr/local/nagios/etc/objects
```
configuration layout:  
services cmds hosts contacts templates.cfg
######templates
```
define service{
service_description ke_template
register 0  //just a template
}
define service{
service_description realservice
use ke_template  //use
}
```
host group:
```
define hostgroup{
  hostgroup_name host
  members a,b,c
}
define service{
  service_description xx
  use ke_template
  hostgroup_name host
  check_command check_ping
}
######installation
