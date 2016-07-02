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
```
######installation
[installation page](https://www.nagios.org/downloads/nagios-core/thanks/?t=1467497693) 4.1.1
```
apt-get install gcc g++ make unzip -y
```
```
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
```
create user
```
adduser --system --no-create-home --disabled-login --group nagios
groupadd nagcmd
usermod -G nagcmd,www-data nagios
id nagios
tar zxvf nagios-4.1.1.tar.gz
cd nagios-4.1.1/
./configure -h  (help)
./configure --prefix=/usr/local/nagios-4.1.1 --with-command-group=nagcmd
make all install
make install-init && make install-config && make install-commandmode
ln -s /usr/local/nagios-4.1.1/ /usr/local/nagios
ln -s /usr/local/nagios/etc/ /etc/nagios
mkdir /var/log/nagios
chown nagios:nagios /var/log/nagios
vim /etc/nagios/nagios.cfg
```
edit
```
log_file=/var/log/nagios/nagios.log
```
then
```
vim /etc/nagios/objects/contacts.cfg
```
edit
```
contant_name ke
members ke
```
then untar plugin
```
tar zxvf nagios-plugins-2.1.1.tar.gz
cd nagios-plugins-2.1.1
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make install
```
check config
```
/usr/local/nagios/bin/nagios -v /etc/nagios/nagios.cfg
```
start service
```
service nagios start
```
```
