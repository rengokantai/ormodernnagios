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
apt-get install gcc g++ make unzip php5-fpm spawn-fcgi fcgiwrap libgd2-xpm-dev -y
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
contant_name root
members root
```
or
```
:%s/nagiosadmin/root/g
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
service fcgiwrap restart
service php5-fpm restart
```
configure nginx
```
vim /etc/nginx/sites-available/nagios.busycorp.com.conf
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/nagios.busycorp.com.conf /etc/nginx/sites-enabled
```

create password
```
P=$(openssl passwd -crypt)
echo $P
echo "root:${P}" >> /etc/nagios/htpasswd.users
```
last step
```
service nginx reload

#grant cgi permissions to the UI
sudo vim /etc/nagios/cgi.cfg
:%s/nagiosadmin/root/g
```
nginx.conf
```
server {
    server_name nagios.busycorp.com;
    access_log /var/log/nginx/nagios.busycorp.com-access.log;
    error_log /var/log/nginx/nagios.busycorp.com-error.log;
 
    auth_basic "Private";
    auth_basic_user_file /etc/nagios/htpasswd.users;
 
    root /var/www/virtualhosts/nagios.busycorp.com;
    index index.php index.html;
 
    location / {
        try_files $uri $uri/ index.php /nagios;
    }
 
    location /nagios {
        alias /usr/local/nagios/share;
    }
 
    location ~ ^/nagios/(.*\.php)$ {
        alias /usr/local/nagios/share/$1;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
 
    location ~ \.cgi$ {
            root /usr/local/nagios/sbin/;
            rewrite ^/nagios/cgi-bin/(.*)\.cgi /$1.cgi break;
            fastcgi_param AUTH_USER $remote_user;
            fastcgi_param REMOTE_USER $remote_user;
            include /etc/nginx/fastcgi_params;
            fastcgi_pass unix:/var/run/fcgiwrap.socket;
      }
 
    location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
      }
}

```
